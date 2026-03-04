---
layout: post
title: "how does my container get a root fs?"
date: 2026-03-02
permalink: images-to-fs
categories: computers
---

this is part 2 of a two-part series on how container images and filesystems work:

1. what is a container image?
2. how does my container get a root filesystem?

## intro

in part 1, we built a two-layer OCI image by hand, imported it into containerd, and ran it. we saw that a container image is an ordered list of filesystem layers plus metadata, and that the container runtime merges them into a single view using overlayfs.

in this post, we'll go deeper. we'll build two images that *share* a base layer, then trace exactly how containerd unpacks them — the overlay mount mechanics, the prepare-apply-commit loop, writable layers, and `pivot_root`. by the end, you'll understand the full chain from a downloaded image to a running container's root filesystem.

the reason containers use these layered filesystems is sharing. lots of containers might use the same base (like ubuntu), and with layers that base only needs to exist on disk once. the ordering of the lower layers matters — it's a list, not a set. we'll refer to layers having "parent" layers.

the container runtime first figures out how to extract the layers and how they relate to one another.

then, when it's ready to create a running container, it creates an overlay mount by passing in a writable layer plus all the read-only layers. this tells the kernel to treat all the layers as part of the same overlay filesystem, so when you look at it, it looks like a "normal" filesystem — the kernel stitches it together for you behind the scenes.

finally, the container runtime "pivots" the root within the container to this newly created overlay mount. because of this, when you enter a container, all you see is the unified filesystem view, and the view of the host's filesystem is gone.

we'll look at all of this through hands-on demos below.

## table of contents

0. [prerequisites](#0-prerequisites)
1. [why a union filesystem?](#1-why-a-union-filesystem)
2. [containerd components](#2-containerd-components)
3. [hands-on with overlay mounts](#3-hands-on-with-overlay-mounts)
4. [the prepare-apply-commit loop](#4-the-prepare-apply-commit-loop)
5. [writable layer](#5-writable-layer)
6. [pivot_root](#6-pivot_root)
7. [summary](#summary)
a. [appendix a: layer sharing](#appendix-a-layer-sharing)
b. [appendix b: volumes](#appendix-b-volumes)

## 0. prerequisites

same setup as part 1 — you need a linux machine with containerd, docker, ctr, jq, and tree. see the [demo-instance-cdk](https://github.com/your-repo/demo-instance-cdk) for a preconfigured environment.

### build & import two images

to demonstrate layer sharing and the unpack loop, we need two images that share a base layer. i'll make one, and my bff dasha will make one. dasha's is a little less fancy with only two layers (sorry dash) but we'll use it to demonstrate how we can share layers.

**annie's image** (3 layers — busybox base + 2 RUN layers):

```dockerfile
FROM busybox

RUN echo "hello from layer 2" > /hello.txt \
  && mkdir -p /data \
  && echo "layer2 config" > /data/config.txt

RUN echo "hello from layer 3" > /hello.txt \
  && echo "i only exist in layer 3" > /layer3.txt
```

**dasha's image** (2 layers — same busybox base + 1 RUN layer):

```dockerfile
FROM busybox

RUN echo "hi dasha" > /hi-dasha.txt
```

both images share the same `busybox` base layer. let's build and import them:

```bash
export WORKDIR=~/container-demo
export CTR_NAMESPACE=spelunking

# build with docker
docker build -t annies-image -f Dockerfile.annie .
docker build -t dashas-image -f Dockerfile.dasha .

# export as tarballs
docker save annies-image -o "$WORKDIR/annies-image.tar"
docker save dashas-image -o "$WORKDIR/dashas-image.tar"

# import annie's image into containerd
# (we'll import dasha's later for the layer sharing demo)
sudo ctr -n "$CTR_NAMESPACE" images import "$WORKDIR/annies-image.tar"

# verify
sudo ctr -n "$CTR_NAMESPACE" images ls
```

## 1. why a union filesystem?

union fs let containers share layers.

if you have containers that share the same base layers, you can re-use them directly. this saves on network bandwidth and disk, and lets containers startup faster once they hit a node.

as far as i can tell, this was the default choice for container filesystems from the get-go. it is, however, not the only option. containerd supports pluggable "snapshotters" — you could use one that doesn't do layering at all (like the `native` snapshotter, which just copies files).

the downside: because layers are shared, you can't just untar everything into a single directory and call it done. you need machinery to track which layers exist, how they relate to each other, and how to mount them. that's what containerd's unpack pipeline does.

## 2. containerd components

containerd's image-to-filesystem pipeline has a few key components:

```
┌─────────────────────────────────────────────────────────┐
│                     content store                       │
│              (raw blobs: tarballs, configs)             │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
              ┌──────────────────┐
              │     unpacker     │
              │                  │
              │  for each layer: │
              │  ┌─► prepare ───┐│
              │  │   apply      ││
              │  │   commit ◄───┘│
              │  └───────────────│
              └────────┬─────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    snapshotter                          │
│          (unpacked layer dirs on disk, chained          │
│           via parent relationships)                     │
└─────────────────────────────────────────────────────────┘
```

**content store**: where downloaded blobs live. the raw layer tarballs, image configs, and manifests, all stored by digest. this is the "what was downloaded" storage.

**snapshotter**: manages the "what's on disk" storage — one directory per unpacked layer. the overlayfs snapshotter stores them under `/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/`. each snapshot knows its parent, forming a list that mirrors the image's layer ordering.

**unpacker + applier**: the orchestration logic that reads blobs from the content store and unpacks them into snapshots. for each layer, it runs the prepare-apply-commit loop (more on this below).

at this point, no layers are mounted or merged. the snapshots are just directories on disk. the overlay mount that creates the unified view happens later, when you actually run a container.

## 3. hands-on with overlay mounts

before we trace containerd's unpack loop, let's build an overlay mount from scratch — no containers, just raw linux filesystem calls.

### what is overlayfs?

overlayfs is a kernel filesystem that layers directories on top of each other. you give it a stack of read-only "lower" directories and one writable "upper" directory, and it presents a "merged" directory that looks like all of them combined.

### build a tiny overlay

```bash
# create the directories
OVERLAY_DIR=$(mktemp -d)
mkdir -p "$OVERLAY_DIR"/{lower1,lower2,upper,work,merged}

# populate the lower layers
echo "from lower1" > "$OVERLAY_DIR/lower1/unique-to-lower1.txt"
echo "from lower1" > "$OVERLAY_DIR/lower1/shared.txt"

echo "from lower2" > "$OVERLAY_DIR/lower2/unique-to-lower2.txt"
echo "from lower2" > "$OVERLAY_DIR/lower2/shared.txt"  # shadows lower1's version

# mount the overlay
sudo mount -t overlay overlay \
  -o "lowerdir=$OVERLAY_DIR/lower2:$OVERLAY_DIR/lower1,upperdir=$OVERLAY_DIR/upper,workdir=$OVERLAY_DIR/work" \
  "$OVERLAY_DIR/merged"
```

note that `lowerdir` lists directories from top to bottom — `lower2` takes priority over `lower1`.

```
before any writes:

┌─────────────────────────────────────────────────────┐
│  merged (mount point)                               │
│    unique-to-lower1.txt = "from lower1"             │
│    unique-to-lower2.txt = "from lower2"             │
│    shared.txt           = "from lower2"             │  (lower2 shadows lower1)
├─────────────────────────────────────────────────────┤
│  upper (empty)                                      │
├─────────────────────────────────────────────────────┤
│  lower2: unique-to-lower2.txt, shared.txt           │
├─────────────────────────────────────────────────────┤
│  lower1: unique-to-lower1.txt, shared.txt           │
└─────────────────────────────────────────────────────┘
```

### explore: reads, writes, shadowing

#### reading: files from both lowers are visible

the kernel checks each layer from top to bottom until it finds the file:

```
read unique-to-lower1.txt:            read unique-to-lower2.txt:
  upper  (miss)                         upper  (miss)
  lower2 (miss)                         lower2 (hit!) → "from lower2"
  lower1 (hit!) → "from lower1"
```

```bash
cat "$OVERLAY_DIR/merged/unique-to-lower1.txt"   # "from lower1"
cat "$OVERLAY_DIR/merged/unique-to-lower2.txt"   # "from lower2"
```

#### shadowing: the topmost layer wins

`shared.txt` exists in both lower1 and lower2. the kernel finds lower2's copy first and stops looking:

```
read shared.txt:
  upper  (miss)
  lower2 (hit!) → "from lower2"
  lower1 (has it, but never reached)
```

```bash
cat "$OVERLAY_DIR/merged/shared.txt"   # "from lower2"
```

#### writing a new file: goes to the upper layer

new files are always created in the writable upper layer:

```
write new-file.txt:
  upper  ← "new file" (created here)
  lower2 (untouched)
  lower1 (untouched)
```

```bash
echo "new file" > "$OVERLAY_DIR/merged/new-file.txt"
ls "$OVERLAY_DIR/upper/"               # new-file.txt appears here
```

#### modifying a lower file: copy-up

when you modify a file that lives in a lower layer, the kernel copies it up to upper first, then modifies the copy. the lower original is untouched:

```
modify unique-to-lower1.txt:
  upper  ← "modified" (copied up, then modified)
  lower2 (untouched)
  lower1 unique-to-lower1.txt = "from lower1" (still intact!)
```

```bash
echo "modified" > "$OVERLAY_DIR/merged/unique-to-lower1.txt"
cat "$OVERLAY_DIR/upper/unique-to-lower1.txt"    # "modified" (copy-up happened)
cat "$OVERLAY_DIR/lower1/unique-to-lower1.txt"   # "from lower1" (unchanged!)
```

#### deleting: creates a whiteout in upper

deleting a file doesn't remove it from the lower layer. instead, the kernel creates a whiteout marker in upper that hides it from the merged view:

```
delete unique-to-lower2.txt:
  upper  ← .wh.unique-to-lower2.txt (whiteout marker)
  lower2 unique-to-lower2.txt = "from lower2" (still intact!)
  lower1 (untouched)
```

```bash
rm "$OVERLAY_DIR/merged/unique-to-lower2.txt"
ls -la "$OVERLAY_DIR/upper/"           # .wh.unique-to-lower2.txt (whiteout marker)
ls "$OVERLAY_DIR/merged/"              # unique-to-lower2.txt is gone from merged view
```

```
after writes:

┌─────────────────────────────────────────────────────┐
│  merged (mount point)                               │
│    unique-to-lower1.txt = "modified"                │  (from upper, copy-up)
│    shared.txt           = "from lower2"             │
│    new-file.txt         = "new file"                │  (from upper)
│    (unique-to-lower2.txt is gone)                   │
├─────────────────────────────────────────────────────┤
│  upper:                                             │
│    unique-to-lower1.txt      "modified"             │
│    new-file.txt              "new file"             │
│    .wh.unique-to-lower2.txt  (whiteout)             │
├─────────────────────────────────────────────────────┤
│  lower2: unique-to-lower2.txt, shared.txt           │  (untouched)
├─────────────────────────────────────────────────────┤
│  lower1: unique-to-lower1.txt, shared.txt           │  (untouched)
└─────────────────────────────────────────────────────┘
```

key takeaways:
- **reads** fall through: the kernel checks upper first, then lower2, then lower1
- **writes** always go to the upper layer
- **modifying** a lower file triggers a "copy-up" — the file is copied to upper, then modified there. the lower original is untouched
- **deleting** creates a whiteout marker in upper. the lower file still exists, but the merged view hides it

cleanup:

```bash
sudo umount "$OVERLAY_DIR/merged"
rm -rf "$OVERLAY_DIR"
```

### how this relates to containers

containerd's snapshotter unpacks each image layer into its own directory under `snapshots/<n>/fs/`. these become the lowerdirs. when a container starts, containerd creates one more directory as the writable upper layer, then mounts everything together.

one detail: if an image has only one layer, containerd uses a bind mount instead of an overlay mount, since there's nothing to merge.

## 4. the prepare-apply-commit loop

when containerd imports an image, it unpacks each layer through a three-step loop:

```
for each layer in the image:

  ┌──────────┐      ┌─────────┐      ┌─────────┐
  │ prepare  │─────►│  apply  │─────►│ commit  │
  │          │      │         │      │         │
  │ create a │      │ untar   │      │ mark as │
  │ staging  │      │ layer   │      │ ready   │
  │ dir      │      │ into it │      │         │
  └──────────┘      └─────────┘      └─────────┘
```

let's define each step:

**prepare**: the snapshotter creates a new staging directory. if this layer has a parent (i.e., it's not the base layer), the staging directory is set up with the parent's snapshot as its lower layer — so the apply step can see files from previous layers. this matters because layers are diffs, so you might need to inherit say directory permissions from your parent layer.

**apply**: the applier untars the layer blob from the content store into the prepared directory. since the directory has visibility into parent layers (via the overlay or bind mount from prepare), the untar can handle things like file ownership inherited from parent layers.

**commit**: the snapshotter marks the snapshot as committed (read-only). this is a containerd state transition — the staging directory becomes a permanent, immutable snapshot that can be used as a parent for the next layer. note! this "immutable committed snapshot" is a containerd application-level concept: it means containerd will not mutate that directory anymore. that immutability isn't enforced at the filesystem level, the directory is the same as it was in the previous step.

this loop runs once per layer, building up the snapshot chain:

```
layer 0 (base):
  prepare → apply busybox.tar → commit
  result: snapshot 1 (busybox files)

layer 1:
  prepare(parent=snapshot 1) → apply layer2.tar → commit
  result: snapshot 2 (layer 2 files, parent=1)

layer 2:
  prepare(parent=snapshot 2) → apply layer3.tar → commit
  result: snapshot 3 (layer 3 files, parent=2)
```

### inspect the results

after importing annie's 3-layer image, let's look at what containerd produced:

```bash
# list content store blobs
sudo ctr -n "$CTR_NAMESPACE" content ls

# list snapshots — notice the parent chain
sudo ctr -n "$CTR_NAMESPACE" snapshots info <snapshot-name>

# look at the snapshot directories on disk
SNAPSHOTS_ROOT="/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots"
sudo ls "$SNAPSHOTS_ROOT"

# each snapshot has an fs/ directory with the unpacked layer
for snap in $(sudo ls "$SNAPSHOTS_ROOT"); do
  echo "--- snapshot $snap ---"
  sudo ls "$SNAPSHOTS_ROOT/$snap/fs/"
  if sudo test -f "$SNAPSHOTS_ROOT/$snap/fs/hello.txt"; then
    echo "hello.txt = $(sudo cat "$SNAPSHOTS_ROOT/$snap/fs/hello.txt")"
  fi
done
```

none of these snapshots are mounted yet. they're just directories. the mounting happens when we start a container.

## 5. writable layer

when containerd starts a container, it adds one more layer on top: the **writable layer** (also called the "active" snapshot or upper directory).

```
┌──────────────────────────────────────┐
│          merged view (rootfs)        │
├──────────────────────────────────────┤
│  upperdir (writable, active snapshot)│  ← container writes go here
├──────────────────────────────────────┤
│  lowerdir[2]: layer 3 snapshot       │  (read-only, committed)
├──────────────────────────────────────┤
│  lowerdir[1]: layer 2 snapshot       │  (read-only, committed)
├──────────────────────────────────────┤
│  lowerdir[0]: busybox snapshot       │  (read-only, committed)
└──────────────────────────────────────┘
```

let's see this in action. before we start the container, we can preview what containerd is about to do. `ctr snapshots prepare` creates the writable layer on top of the committed snapshot chain, and `ctr snapshots mounts` shows us the exact overlay mount command the runtime will use:

```bash
# get the top layer's chain ID (the snapshot name for the topmost committed layer)
TOP_SNAPSHOT=$(sudo ctr -n "$CTR_NAMESPACE" snapshots ls | tail -1 | awk '{print $1}')

# prepare a writable layer on top of the committed chain
sudo ctr -n "$CTR_NAMESPACE" snapshots prepare demo-active "$TOP_SNAPSHOT"

# see what the overlay mount will look like
sudo ctr -n "$CTR_NAMESPACE" snapshots mounts /tmp/demo-mountpoint demo-active
```

the `mounts` command prints the exact `mount -t overlay` invocation containerd will use — you can see the lowerdirs (the committed snapshots) and the upperdir (the new writable layer). this is exactly what happens behind the scenes when `ctr run` starts a container.

let's clean up that preview and do it for real:

```bash
sudo ctr -n "$CTR_NAMESPACE" snapshots rm demo-active
```

now let's start the container:

```bash
# run a container in the background
sudo ctr -n "$CTR_NAMESPACE" run -d docker.io/library/annies-image:latest demo-annie /bin/sh -c "sleep 3600"

# find the overlay mount on the host
mount | grep overlay | grep "$CTR_NAMESPACE"
# or:
sudo cat /proc/$(sudo ctr -n "$CTR_NAMESPACE" tasks ls -q | head -1)/mountinfo | grep overlay
```

you'll see the mount with `upperdir=<path>` — that's the writable layer. let's write a file from inside the container and find it on the host:

```bash
# write a file inside the container
sudo ctr -n "$CTR_NAMESPACE" tasks exec --exec-id test demo-annie /bin/sh -c "echo 'written at runtime' > /runtime-file.txt"

# find it in the upper directory on the host
UPPERDIR=$(mount | grep overlay | grep "$CTR_NAMESPACE" | grep -oP 'upperdir=\K[^,]+')
sudo cat "$UPPERDIR/runtime-file.txt"
# -> written at runtime
```

the file only exists in the upper directory. the lower snapshots are untouched.

now kill the container:

```bash
sudo ctr -n "$CTR_NAMESPACE" tasks kill demo-annie
sudo ctr -n "$CTR_NAMESPACE" containers rm demo-annie
```

the writable layer is gone. we can verify — the upperdir we found earlier no longer exists:

```bash
sudo ls "$UPPERDIR" 2>&1
# -> ls: cannot access '...': No such file or directory
```

but the read-only snapshots are still there — they belong to the *image*, not the container:

```bash
SNAPSHOTS_ROOT="/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots"
sudo ls "$SNAPSHOTS_ROOT"

# verify snapshot contents are still intact
for snap in $(sudo ls "$SNAPSHOTS_ROOT"); do
  if sudo test -f "$SNAPSHOTS_ROOT/$snap/fs/hello.txt"; then
    echo "snapshot $snap: $(sudo cat "$SNAPSHOTS_ROOT/$snap/fs/hello.txt")"
  fi
done
```

the committed snapshots stick around as long as the image is imported. only the writable upper layer is ephemeral — it lives and dies with the container.

that's why when you write a file inside a container — say, in the root directory — it doesn't persist after the container stops. the upper directory is tied to the container's lifetime, while the lower layers are tied to the image's lifetime.

## 6. pivot_root

we have our overlay mount producing a merged filesystem. but when you exec into a container, that merged view is *all you see*. the host's filesystem is completely gone. what's up with that?

the answer is **`pivot_root`**.

`pivot_root` is a linux syscall that swaps the root filesystem of a process's mount namespace. the container runtime:

1. creates a new mount namespace for the container (via `unshare` or `clone`)
2. mounts the overlay at a temporary location
3. calls `pivot_root` to make the overlay mount the new `/`
4. unmounts the old root

```
before pivot_root:
┌─────────────────────────────┐
│  /  (host root)             │
│  ├── /home/...              │
│  ├── /var/lib/containerd/...│
│  └── /tmp/container-root/   │  ← overlay mounted here
│       ├── bin/              │
│       ├── hello.txt         │
│       └── ...               │
└─────────────────────────────┘

after pivot_root:
┌─────────────────────────────┐
│  /  (container root)        │  ← was /tmp/container-root/
│  ├── bin/                   │
│  ├── hello.txt              │
│  └── ...                    │
│  (host root is gone!)       │
└─────────────────────────────┘
```

### see it in action

from inside a running container, you can verify the overlay mount is the root:

```bash
# start a container
sudo ctr -n "$CTR_NAMESPACE" run --rm -t docker.io/library/annies-image:latest demo-annie /bin/sh

# inside the container:
cat /proc/1/mountinfo | head -5
```

you'll see that `/` is an overlay mount. the container has no visibility into the host's filesystem — `pivot_root` made the overlay the entire world.

from the **host**, you can contrast this with the host's view:

```bash
cat /proc/1/mountinfo | head -5
```

the host's PID 1 has a completely different set of mounts. the container's mount namespace is isolated.

but the host *can* still peek into the container's root filesystem — the kernel exposes it via `/proc/<pid>/root`:

```bash
# from the host, find the container's PID
TASK_PID=$(sudo ctr -n "$CTR_NAMESPACE" tasks ls | grep demo-annie | awk '{print $2}')

# peek into the container's root from the host
# this is EXACTLY what we see when we exec into the container
sudo ls /proc/$TASK_PID/root/
# -> bin/  data/  dev/  etc/  hello.txt  layer3.txt  proc/  sys/

sudo cat /proc/$TASK_PID/root/hello.txt
# -> hello from layer 3
```

this is the same merged overlay view the container sees as `/`. the kernel just lets the host access it through the proc filesystem. the container itself has no idea — from its perspective, `pivot_root` made the overlay the entire world.

## summary

we've now traced the full path from a container image to a running container's root filesystem:

```
registry / docker save
        │
        ▼
┌───────────────────────┐
│    content store      │   blobs stored by digest
└───────────┬───────────┘
            │
    prepare / apply / commit
    (once per layer)
            │
            ▼
┌───────────────────────┐
│     snapshotter       │   one directory per layer,
│                       │   chained via parents
└───────────┬───────────┘
            │
    overlay mount
    (lowerdirs + upperdir)
            │
            ▼
┌───────────────────────┐
│    merged rootfs      │   single unified view
└───────────┬───────────┘
            │
    pivot_root
            │
            ▼
┌───────────────────────┐
│  running container    │   overlay is now /
│  process              │
└───────────────────────┘
```

containers don't have their own copy of a filesystem. this is a way in which containers differ from VMs: they just have a partitioned view of the host. they have a view of the filesystem — a union of shared, read-only layers plus one ephemeral writable layer, pivoted to become the process's root. the machinery exists to make sharing efficient and to make the layering invisible to the process inside.

## appendix a: layer sharing

annie's image is already imported. let's see how many snapshots it created, then import dasha's and watch sharing in action:

```bash
# check current state — only annie's image
sudo ctr -n "$CTR_NAMESPACE" images ls
sudo ctr -n "$CTR_NAMESPACE" snapshots ls

# now import dasha's image — she shares the same busybox base layer
sudo ctr -n "$CTR_NAMESPACE" images import "$WORKDIR/dashas-image.tar"

# check snapshots again
sudo ctr -n "$CTR_NAMESPACE" snapshots ls
```

you'll notice that importing dasha's 2-layer image created only **one** new snapshot, not two. the busybox base snapshot already existed from annie's import, so containerd reused it. both images reference the same snapshot as their base layer, because the layer content (and therefore its DiffID) is identical.

```
annie's image:                  dasha's image:

  snapshot 3 (layer 3)
       │
  snapshot 2 (layer 2)            snapshot 4 (dasha's layer)
       │                               │
       └──────────┬────────────────────┘
                  │
            snapshot 1 (busybox base)     ← SHARED
```

the content store is also deduplicated — the busybox layer blob is stored only once.

```bash
SNAPSHOTS_ROOT="/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots"
sudo ls "$SNAPSHOTS_ROOT"
# you'll see snapshots for each unique layer, not each image
```

this is why layer ordering matters and why the spec calls layers "diffs." a layer isn't a complete filesystem — it's a delta relative to its parent. the same delta only makes sense if applied on top of the same parent chain. that's why containerd tracks parent relationships, and why two images can share a layer only if they have the same ancestry up to that point.

## appendix b: volumes

if the writable layer is ephemeral, how do volume mounts persist data?

volumes work differently from the overlay. they're **bind mounts** — a host directory is directly mounted into the container's filesystem at a specific path. reads and writes go straight to the host directory. no overlay, no copy-up, no whiteout markers. so you write to the host, not to ephemeral storage.