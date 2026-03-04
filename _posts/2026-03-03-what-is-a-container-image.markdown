---
layout: post
title: "what is a container image?"
date: 2026-03-02
permalink: what-is-a-container-image
categories: computers
---

this is part 1 of a two-part series on how container images and filesystems work:

1. **what is a container image?** (this post)
2. how does my container get a root filesystem?

## intro

my first mental model of a container was: a container is like when your friend wants to send you some files to run, and you put them in a tiny computer so the files work everywhere.

that's... kind of right, and kind of not. so... if kind of not, then how _do_ container images work?

a container image is not a set of files. it's an ordered list of filesystems, plus some metadata. container runtimes use union filesystems to stack these filesystems on top of each other.

it's like a stack of overhead transparencies: you have several sheets and if you layer them, what gets projected looks like a single unified view. the bottom layers are read-only, and the top layer is writable.

why do container images use layers? why bother? 

the answer is sharing. because the lower layers are read-only, we can share them between containers. 

lots of containers might use the same base image. with layers, that base only needs to exist on disk once, and every container that uses it just stacks its own changes on top.

in this post, we'll build a minimal two-layer OCI image entirely by hand — no docker, no buildkit — import it into containerd, inspect the internals, and run it. in part 2, we'll see how a container image gets unpacked into the root filesystem of a running container, and see how the layer sharing works in practice.

## table of contents

0. [prerequisites](#0-prerequisites)
1. [what is a container image?](#1-what-is-a-container-image)
2. [create the filesystem for each layer](#2-create-the-filesystem-for-each-layer)
3. [package layers as tarballs](#3-package-layers-as-tarballs)
4. [assemble the OCI image layout](#4-assemble-the-oci-image-layout)
5. [import into containerd](#5-import-into-containerd)
6. [overlayfs, snapshots & mounts](#6-overlayfs-snapshots--mounts)
7. [inspect containerd state](#7-inspect-containerd-state)
8. [run the container](#8-run-the-container)
9. [summary](#summary)
10. [appendix](#appendix)

## 0. prerequisites

to follow along, you'll need a linux machine with:

- **containerd** and **runc** (the container runtime)
- **ctr** (containerd's CLI)
- **jq**, **tree**, **tar**, **gzip**, **sha256sum**, **stat**
- a **statically-linked busybox** binary (`busybox-static` on most distros)

let's set up a working directory and a containerd namespace so we can clean up easily afterwards:

```bash
export WORKDIR=~/container-demo
export CTR_NAMESPACE=spelunking
mkdir -p "$WORKDIR"
```

all `ctr` commands in this post use `--namespace $CTR_NAMESPACE` (or the shorthand `-n`). this keeps our experiments isolated from anything else running on the machine.

## 1. what is a container image?

a container image is **not** a single blob of files. it's a structured bundle of **layer filesystems** plus **metadata** describing how they fit together. the [OCI image spec](https://github.com/opencontainers/image-spec) defines the format, and it looks like this:

```
index.json
  └─► manifest
        ├─► config            (diffIDs, cmd, env, ...)
        ├─► layer[0] blob     (base.tar.gz)
        └─► layer[1] blob     (delta.tar.gz)
```

let's define those pieces:

- **layers** are tar archives, each containing a filesystem tree. they stack in order — layer 0 is the base, layer 1 is applied on top, and so on.
- the **config** describes how to run the image (command, environment variables, working directory) and lists the **diffIDs** — the sha256 hash of each layer's *uncompressed* tar. diffIDs identify the layer content itself, regardless of compression.
- the **manifest** ties config + layers together. it references each blob by its **digest** — the sha256 hash of the blob *as stored* (usually compressed). it also records the size of each blob.
- the **index** (also called the "image index") is the top-level entry point. it points to one or more manifests (one per platform/architecture).

everything in an OCI image is **content-addressed**: stored and referenced by its sha256 hash. this means you can verify integrity at every level — if a blob's hash doesn't match its expected digest, something is wrong.

we're going to build all of these pieces by hand.

## 2. create the filesystem for each layer

our image will have two layers:

- **base layer**: a minimal filesystem with a statically-linked busybox, a `hello.txt` file, and a config file
- **delta layer**: overrides `hello.txt` (demonstrating layer shadowing) and adds a whiteout marker to delete the config file (demonstrating layer deletion)

here's the equivalent Dockerfile for what we're about to do by hand:

```dockerfile
FROM scratch AS base
COPY build/base/ /

FROM base
COPY build/delta/ /
```

let's build the directory trees:

```bash
# create build directories
mkdir -p "$WORKDIR/build/base" "$WORKDIR/build/delta"

# --- base layer ---
# add busybox (our "distro")
mkdir -p "$WORKDIR/build/base/bin"
cp /bin/busybox "$WORKDIR/build/base/bin/busybox"

# symlink the commands we need to busybox
for cmd in sh ls cat; do
  ln -sf busybox "$WORKDIR/build/base/bin/$cmd"
done

# add some files
echo "hello from base" > "$WORKDIR/build/base/hello.txt"
mkdir -p "$WORKDIR/build/base/data"
echo "base config" > "$WORKDIR/build/base/data/config.txt"

# --- delta layer ---
# shadow hello.txt with new content
echo "hello from delta" > "$WORKDIR/build/delta/hello.txt"

# whiteout marker: tells the runtime to delete data/config.txt
mkdir -p "$WORKDIR/build/delta/data"
sudo mknod "$WORKDIR/build/delta/data/.wh.config.txt" c 0 0
```

let's verify the layout:

```bash
tree "$WORKDIR/build"
```

```
build/
├── base/
│   ├── bin/
│   │   ├── busybox
│   │   ├── cat -> busybox
│   │   ├── ls -> busybox
│   │   └── sh -> busybox
│   ├── data/
│   │   └── config.txt        ("base config")
│   └── hello.txt              ("hello from base")
└── delta/
    ├── data/
    │   └── .wh.config.txt     (whiteout marker — deletes config.txt)
    └── hello.txt              ("hello from delta" — shadows base)
```

notice two important things in the delta layer:

1. **`hello.txt`** exists in both layers. when these layers are stacked, the delta's version will *shadow* the base's version — just like a transparency placed on top of another.
2. **`.wh.config.txt`** is a whiteout file. the `.wh.` prefix is a convention defined in the OCI spec. it tells the runtime: 'in the merged view, pretend `config.txt` doesn't exist.' the file in the base layer isn't actually deleted — it's just hidden.

## 3. package layers as tarballs

each layer in an OCI image is a tar archive (usually gzip-compressed). two hashes matter:

```
files ──► tar ──► sha256 = DiffID ──► gzip ──► sha256 = Digest
                  (uncompressed)                (compressed)
```

- the **DiffID** is the sha256 of the *uncompressed* tar. this is what goes in the image config.
- the **Digest** is the sha256 of the *compressed* tar (the blob as stored). this is what the manifest uses to reference blobs.

let's create reproducible tarballs. we pin `--mtime` and `--owner` so the archives are deterministic — same input always produces the same hash:

```bash
# create tar archives (uncompressed)
tar -C "$WORKDIR/build/base" \
  --sort=name --mtime="2025-01-01 00:00:00" \
  --owner=0 --group=0 --numeric-owner \
  -cf "$WORKDIR/base-layer.tar" .

tar -C "$WORKDIR/build/delta" \
  --sort=name --mtime="2025-01-01 00:00:00" \
  --owner=0 --group=0 --numeric-owner \
  -cf "$WORKDIR/delta-layer.tar" .

# compute DiffIDs (sha256 of uncompressed tar)
BASE_DIFFID="sha256:$(sha256sum "$WORKDIR/base-layer.tar" | cut -d' ' -f1)"
DELTA_DIFFID="sha256:$(sha256sum "$WORKDIR/delta-layer.tar" | cut -d' ' -f1)"

# compress
gzip -kf "$WORKDIR/base-layer.tar"
gzip -kf "$WORKDIR/delta-layer.tar"

# compute Digests (sha256 of compressed tar)
BASE_DIGEST="sha256:$(sha256sum "$WORKDIR/base-layer.tar.gz" | cut -d' ' -f1)"
DELTA_DIGEST="sha256:$(sha256sum "$WORKDIR/delta-layer.tar.gz" | cut -d' ' -f1)"

# record sizes (needed for the manifest)
BASE_SIZE=$(stat -c%s "$WORKDIR/base-layer.tar.gz")
DELTA_SIZE=$(stat -c%s "$WORKDIR/delta-layer.tar.gz")

echo "base  DiffID: $BASE_DIFFID"
echo "base  Digest: $BASE_DIGEST  Size: $BASE_SIZE"
echo "delta DiffID: $DELTA_DIFFID"
echo "delta Digest: $DELTA_DIGEST  Size: $DELTA_SIZE"
```

we now have four hashes — two DiffIDs and two Digests. we'll use them in the next step to wire everything together.

## 4. assemble the OCI image layout

an OCI image on disk is just a directory tree with a specific structure. we need to:

1. write the `oci-layout` marker file
2. place layer blobs in `blobs/sha256/`
3. create the image config
4. create the manifest
5. create the index

```bash
# initialize the OCI layout directory
OCI_DIR="$WORKDIR/oci"
mkdir -p "$OCI_DIR/blobs/sha256"

echo '{"imageLayoutVersion": "1.0.0"}' > "$OCI_DIR/oci-layout"
```

### place the layer blobs

blobs are stored by their digest. the filename is just the hash (without the `sha256:` prefix):

```bash
cp "$WORKDIR/base-layer.tar.gz" "$OCI_DIR/blobs/sha256/${BASE_DIGEST#sha256:}"
cp "$WORKDIR/delta-layer.tar.gz" "$OCI_DIR/blobs/sha256/${DELTA_DIGEST#sha256:}"
```

### create the config

the config describes runtime settings and lists the layer DiffIDs (uncompressed hashes):

```bash
CONFIG=$(jq -n \
  --arg base_diffid "$BASE_DIFFID" \
  --arg delta_diffid "$DELTA_DIFFID" \
  '{
    architecture: "amd64",
    os: "linux",
    rootfs: {
      type: "layers",
      diff_ids: [$base_diffid, $delta_diffid]
    },
    config: {
      Cmd: ["/bin/sh"]
    }
  }')

# store config blob by its digest
CONFIG_DIGEST="sha256:$(echo "$CONFIG" | sha256sum | cut -d' ' -f1)"
CONFIG_SIZE=$(echo "$CONFIG" | wc -c | tr -d ' ')
echo "$CONFIG" > "$OCI_DIR/blobs/sha256/${CONFIG_DIGEST#sha256:}"
```

### create the manifest

the manifest ties the config and layer blobs together, referencing everything by digest:

```bash
MANIFEST=$(jq -n \
  --arg config_digest "$CONFIG_DIGEST" \
  --argjson config_size "$CONFIG_SIZE" \
  --arg base_digest "$BASE_DIGEST" \
  --argjson base_size "$BASE_SIZE" \
  --arg delta_digest "$DELTA_DIGEST" \
  --argjson delta_size "$DELTA_SIZE" \
  '{
    schemaVersion: 2,
    mediaType: "application/vnd.oci.image.manifest.v1+json",
    config: {
      mediaType: "application/vnd.oci.image.config.v1+json",
      digest: $config_digest,
      size: $config_size
    },
    layers: [
      {
        mediaType: "application/vnd.oci.image.layer.v1.tar+gzip",
        digest: $base_digest,
        size: $base_size
      },
      {
        mediaType: "application/vnd.oci.image.layer.v1.tar+gzip",
        digest: $delta_digest,
        size: $delta_size
      }
    ]
  }')

MANIFEST_DIGEST="sha256:$(echo "$MANIFEST" | sha256sum | cut -d' ' -f1)"
MANIFEST_SIZE=$(echo "$MANIFEST" | wc -c | tr -d ' ')
echo "$MANIFEST" > "$OCI_DIR/blobs/sha256/${MANIFEST_DIGEST#sha256:}"
```

### create the index

the index is the top-level entry point. it points to our manifest:

```bash
jq -n \
  --arg manifest_digest "$MANIFEST_DIGEST" \
  --argjson manifest_size "$MANIFEST_SIZE" \
  '{
    schemaVersion: 2,
    manifests: [
      {
        mediaType: "application/vnd.oci.image.manifest.v1+json",
        digest: $manifest_digest,
        size: $manifest_size,
        annotations: {
          "org.opencontainers.image.ref.name": "handroll:latest"
        }
      }
    ]
  }' > "$OCI_DIR/index.json"
```

let's verify the final layout:

```bash
tree "$OCI_DIR"
```

```
oci/
├── blobs/
│   └── sha256/
│       ├── <base layer digest>      (base.tar.gz)
│       ├── <delta layer digest>     (delta.tar.gz)
│       ├── <config digest>          (config JSON)
│       └── <manifest digest>        (manifest JSON)
├── index.json
└── oci-layout
```

everything is content-addressed. the index points to the manifest by digest, the manifest points to the config and layers by digest. you can verify any blob by hashing it and comparing to its expected digest.

## 5. import into containerd

now let's import our hand-built image into containerd. `ctr images import` expects a tar archive of the OCI layout directory:

```bash
# create a tarball of the OCI layout
tar -C "$OCI_DIR" -cf "$WORKDIR/handroll-image.tar" .

# import into containerd
sudo ctr -n "$CTR_NAMESPACE" images import --base-name handroll "$WORKDIR/handroll-image.tar"

# verify it's there
sudo ctr -n "$CTR_NAMESPACE" images ls
```

you should see `docker.io/library/handroll:latest` in the output. containerd has stored the blobs in its content store and unpacked the layers into snapshots.

## 6. overlayfs, snapshots & mounts

before we inspect what containerd did, let's build a mental model of how overlayfs works.

overlayfs is a **union filesystem**. it takes a stack of directories and presents them as one merged view:

```
┌──────────────────────────────────────┐
│          merged view (rootfs)        │  ← what the container sees
├──────────────────────────────────────┤
│       upperdir (writable layer)      │  ← runtime writes go here
├──────────────────────────────────────┤
│  lowerdir[1]: delta snapshot         │  ← hello.txt = "hello from delta"
├──────────────────────────────────────┤
│  lowerdir[0]: base snapshot          │  ← hello.txt = "hello from base"
└──────────────────────────────────────┘
```

- **lowerdirs** are read-only. these are the image's layers, unpacked from the tarballs into snapshot directories.
- the **upperdir** is writable. when a container writes a file, it goes here. when the container is destroyed, this directory is deleted — that's why writes inside a container don't persist.
- the **merged** directory is what the container actually sees: a single unified view where the kernel resolves conflicts by picking the topmost layer.

here's the full chain from what we've built so far to a running container:

1. we created filesystem trees (directories with files)
2. we packaged them into tarballs
3. we assembled those tarballs + metadata into an OCI image layout
4. containerd imported the blobs into its **content store** and unpacked them into **snapshot** directories
5. when containerd starts a container, it creates an **overlay mount** with a writable layer on top of the read-only layers
6. the result: a single merged filesystem that looks "normal" to the container

## 7. inspect containerd state

let's see what containerd actually did when we imported the image.

we'll look a LOT more at all these details in part 2.

### content store

the content store holds all the raw blobs — layer tarballs, config, manifest:

```bash
sudo ctr -n "$CTR_NAMESPACE" content ls
```

you'll see entries matching the digests we computed earlier. containerd stored our blobs verbatim.

### snapshots

the snapshotter unpacked the layer tarballs into directories. these unpacked directories are called snapshots. each layer gets its own snapshot, and snapshots have parents. that is just a way to track where in the filesystem list each filesystem is:

```bash
sudo ctr -n "$CTR_NAMESPACE" snapshots ls
```

you should see two snapshots — one for each layer. let's look at them on disk:

```bash
# find the snapshot directories
SNAPSHOTS_ROOT="/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots"
sudo ls "$SNAPSHOTS_ROOT"
```

each numbered directory contains an `fs/` subdirectory with the unpacked layer contents:

```bash
# inspect each snapshot's contents
for snap in $(sudo ls "$SNAPSHOTS_ROOT"); do
  echo "--- snapshot $snap ---"
  sudo ls "$SNAPSHOTS_ROOT/$snap/fs/"
  if sudo test -f "$SNAPSHOTS_ROOT/$snap/fs/hello.txt"; then
    echo "hello.txt = $(sudo cat "$SNAPSHOTS_ROOT/$snap/fs/hello.txt")"
  fi
done
```

the snapshots are just directories on disk with the unpacked layer contents. no overlays yet — that happens when we run a container.

## 8. run the container

let's run a container from our hand-built image and explore what happens:

```bash
sudo ctr -n "$CTR_NAMESPACE" run --rm -t docker.io/library/handroll:latest demo /bin/sh
```

### the merged view

inside the container, you see a single unified filesystem:

```bash
ls /
# -> bin/  data/  dev/  etc/  hello.txt  proc/  sys/
```

it looks like a "normal" filesystem. the layering is invisible.

### layer shadowing

```bash
cat /hello.txt
# -> hello from delta
```

the delta layer's `hello.txt` shadows the base layer's version. the base version still exists on disk in its snapshot directory — it's just hidden in the merged view.

the base layer's other content is still visible:

```bash
ls /bin/
# -> busybox  cat  ls  sh
```

### whiteout deletion

```bash
ls /data/
# (empty — config.txt has been "deleted" by the whiteout marker)
```

remember the `.wh.config.txt` file we created in the delta layer? the container runtime processed it during overlay setup: `data/config.txt` from the base layer is hidden in the merged view. the file is still physically present in the base snapshot — it's just invisible from inside the container.

let's prove that. from the host, look at the base layer's snapshot directory:

```bash
# the base snapshot still has everything
for snap in $(sudo ls "$SNAPSHOTS_ROOT"); do
  if sudo test -f "$SNAPSHOTS_ROOT/$snap/fs/data/config.txt"; then
    sudo cat "$SNAPSHOTS_ROOT/$snap/fs/hello.txt"
    # -> hello from base
    sudo cat "$SNAPSHOTS_ROOT/$snap/fs/data/config.txt"
    # -> base config
  fi
done
```

both files are right there on disk. the overlay just hides them from the container's view.

### inspect the overlay mount

```bash
cat /proc/1/mountinfo | grep overlay
```

you'll see something like:

```
... overlay overlay rw,lowerdir=<snapshot2>/fs:<snapshot1>/fs,upperdir=<active>/fs,workdir=<active>/work ...
```

this is the actual kernel mount that produces the merged view. you can see:
- `lowerdir` lists the read-only snapshots (delta first, then base — order matters!)
- `upperdir` is the writable directory for this container's lifetime
- `workdir` is used internally by overlayfs for atomic operations


## summary

we've traced the full path from raw files to a running container:

```
filesystem trees
    ↓
tarball layers (tar + gzip)
    ↓
OCI image layout (blobs + config + manifest + index)
    ↓
containerd content store (blobs stored by digest)
    ↓
snapshots on disk (layers unpacked into directories)
    ↓
overlayfs mount (lowerdirs + upperdir = merged view)
    ↓
running container process
```

we saw that a container image is just some file trees and metadata. we saw how those are ingested & set up by containerd into a running container.

the interesting beating heart here is that this is all a story about **layered filesystems**. every piece of this chain exists to get a stack of directory trees merged into a single view that a process can use as its root filesystem.

in part 2, we'll look at how all this machinery actually works. we build images with shared layers, trace how containerd's prepare-apply-commit loop unpacks them, inspect writable layers and `pivot_root`, and see how layer sharing saves disk space.

## appendix

### layers are diffs

if a file is byte-for-byte identical in two layers — same content, same metadata — build tooling won't include it in the upper layer's tarball. since we're using overlay mounts, a file in a lower layer is already visible in the merged view, so there's no reason to duplicate it.

each layer only contains what changed relative to the layers below it.

### cleanup

to clean up everything we created:

```bash
# remove the container (if still running)
sudo ctr -n "$CTR_NAMESPACE" tasks kill demo 2>/dev/null
sudo ctr -n "$CTR_NAMESPACE" containers rm demo 2>/dev/null

# remove the image
sudo ctr -n "$CTR_NAMESPACE" images rm docker.io/library/handroll:latest

# remove the working directory
rm -rf "$WORKDIR"
```
