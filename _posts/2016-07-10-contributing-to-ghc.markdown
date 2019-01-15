---
layout: post
title: "Contributing to GHC"
date: 2016-07-10
permalink: projects/contributing-to-ghc
categories: computers
---


I came to the [RC](https://www.recurse.com/) thinking that I would contribute to the Elm compiler... but then abandoned the idea for many reasons of varying validity- but it definitely felt intimidating! A few weeks later I was talking to James, who told me he had made a small contribution to GHC- the Haskell compiler- so over the weekend I ended up for the [newcomer page](https://ghc.haskell.org/trac/ghc/wiki/Newcomers) for GHC. At the bottom of this page was [this post](https://rawgit.com/gibiansky/4c54f767bf21a6954b23/raw/67c62c5555f40c6fb67b124307725df168201361/exp.html) which is a walk through of Andrew's first GHC contribution.

And you know what, if Andrew can do it, I can do it. (Can and did!) Damn the torpedoes, full speed ahead!

So in parallel to Andrew's guide to his first contribution in excruciating detail, here is mine:

---

# Compiling GHC from the source

I followed the newcomer instructions and had *surprisingly few* issues getting GHC to compile. It did take a while though! Here's the rundown:

I ran:

```bash
$ git clone --recursive git://github.com/ghc/ghc
$ cd ghc/
$ cp mk/build.mk.sample mk/build.mk
```

&nbsp;
All of these worked with little excitement. No news is good news! Then I ran:

```bash
$ ./boot
```

and was given the message:

```
Booting libraries/time/
Booting libraries/unix/
Can't exec "aclocal": No such file or directory at /usr/local/Cellar/autoconf/2.69/share/autoconf/Autom4te/FileUtils.pm line 326.
autoreconf: failed to run aclocal: No such file or directory
Can't exec "aclocal": No such file or directory at /usr/local/Cellar/autoconf/2.69/share/autoconf/Autom4te/FileUtils.pm line 326.
autoreconf: failed to run aclocal: No such file or directory
Running autoreconf failed at ./boot line 217, <PKGS> line 12.
```

&nbsp;
The first hit stackoverflow post suggested that I install automake, so I did.

```bash
$ brew install automake
$ ./boot
```

Wooh! It worked! Onward!

```bash
$ ./configure
```

gave me the message:

```
checking for version of happy... 1.19.5
checking for alex... no
checking for version of alex...
configure: error: Alex version 3.1.0 or later is required to compile GHC.
```

&nbsp;
okay, cool, no problem, I ran:

```bash
$ cabal install Alex
$ ./configure
```

and got the message:

```
config.status: creating mk/config.h
\--------------------------------------------------------------------
Configure completed successfully.
   Building GHC version  : 8.1.20160724
          Git commit id  : 4036c1f110578f8e2813295116b79a5a06e2bf59
```

&nbsp;
Score! Go team!
okay, now to make:

%%%%%%   update (02/2017)  %%%%%%

I didn't realize this when I originally built GHC but my build would have gone much faster if I had uncommented `BuildFlavor = devel2` in the mk/build.mk *before* running make.

%%%%%%%%%%%%%%%%%%%%%%%

```bash
$ make -j8 # parallelize to at most 8 parallel jobs; adapt to actual number of cpu cores
```

…wait, how many cores do I have? Quick google search turned up the command: (on osx!)

```bash
$ sysctl -n hw.ncpu
 4
```

Well unfortunately I was too excited and ran make before checking how many cores I had… and I didn't want to interrupt halfway through the build, so I just let it compile...
...and half an hour later had to borrow a laptop charger because I was running down my battery by running at 100% CPU…
About an hour later I took a look at my CPU load:

![](/images/ghc1.jpg)

and found out that, since I was trying to run 8 threads on a 4 core machine, not only were the 8 parallel threads sucking all my CPU but also the OS was taking a ton of CPU to juggle them. Yikes!

I decided to give it until 7 pm, before pulling the plug to try it again overnight with the right parallelization flag.

6:47pm: Compilation finished! I was notified when my computer's fan turned off.

I checked if the compilation *actually* succeeded by running:

```bash
$ ./inplace/bin/ghc-stage2 —interactive
```

And it worked!

```bash
$ ./inplace/bin/ghc-stage2 —interactive
GHCi, version 8.1.20160724: http://www.haskell.org/ghc/  :? for help
Prelude> putStrLn "yay!"
yay!
```

&nbsp;
Yay! Onward!

---

# Verifying I can modify the source

Okay, so before starting in on the issue I made sure I was set up to see any changes that I made actually propagate through the compiler. The issue I was looking at had to do with the "relevant bindings" provided in a typechecker error message, so that's where I went to make my toy change.

First of all, before touching ANYTHING I created a new branch.

```bash
$ git checkout -b typecheck-variable-shadowing
```

Then I also updated the mk/build.mk file as recommended in the newcomers page:

1) I uncommented `BuildFlavor = devel2`

2) also uncommented `stage=2`

I had read that GHC had a wiki, so I thought I'd check out the Typechecker entry before diving in, in case it had any helpful insite:

![](/images/ghc2.jpg)

Um, nevermind. Diving in!
I got over the the typechecker directory:

```bash
$ cd compiler/typecheck
```

and then used ack (like grep, but with pretty-printing) to find what file was producing the type error in question:

```bash
$ ack "Relevant bindings"
```

![](/images/ghc3.jpg)

Stellar. I opened that file and made a minimal change- I just changed the word "bindings" to "donuts". (I opened the typecheck directory in Sublime, and that was pretty convenient. Atom is also has good Haskell support. ) Let's see if it worked!

```bash
$ cd ..; make fast
```

I got this message:

```
libraries/ghci/ghc.mk:4: libraries/ghci/dist-install/build/.depend-v-dyn.c_asm: No such file or directory
Configured with: --prefix=/Applications/Xcode.app/Contents/Developer/usr --with-gxx-include-dir=/usr/include/c++/4.2.1
compiler/ghc.mk:572: compiler/stage2/build/.depend-v-dyn.haskell: No such file or directory
compiler/ghc.mk:572: compiler/stage2/build/.depend-v-dyn.c_asm: No such file or directory
make[2]: *** No rule to make target `compiler/stage2/build/.depend-v-dyn.c_asm'.  Stop.
make[1]: *** [all_compiler] Error 2
make: *** [all] Error 2
```

&nbsp;
Hmmm, okay. So I'll try making without the fast flag:

```bash
$ make
```

And that worked! It took a few minutes, I'm not sure what the issue with make fast was. Okay, let's see if I can see my change:

```bash
$ ../inplace/bin/ghc-stage2 --interactive
```

![](/images/screenshot1.jpg)

&nbsp;
Yeah! We're in business!

---

# Fixing a GHC Bug!

Okay, so I worked on [this bug](https://ghc.haskell.org/trac/ghc/ticket/12177). Here's the premise: when you write a function you can leave a hole (just an underscore _ ) and when you compile GHC will throw an error that will tell you what the type of that hole is. So if you type:

```haskell
Prelude> \x-> "Hello " ++ _
```

ghci will tell you:

```
<interactive>:2:18: error:
    • Found hole: _ :: [Char]
    • In the second argument of ‘(++)’, namely ‘_’
      In the expression: "Hello " ++ _
      In the expression: \ x -> "Hello " ++ _
    • Relevant bindings include
        x :: t (bound at <interactive>:2:2)
        it :: t -> [Char] (bound at <interactive>:2:1)
```

&nbsp;
GHC can infer that the hole needs to have type [Char]. Cool, that could be helpful. So now the bug was that if we have a shadowed variable, then GHC was reporting the outer binding as a "Relevant binding" even though it couldn't possibly be relevant as it was shadowed. That probably didn't make very much sense, so here's an example. I'm going to show what ghci reports, and highlight the offending line in blue:

![](/images/screenshot2.jpg)


&nbsp;

That line in blue is the binding for the outside (left-most) variable 'x' which we can never access in the inner scope because we re-bind x in that inner context. So my task was to get the typechecker to remove that line in blue.

So I don't really have a good summary of what I did next- but in short, I poked around the typechecker for a few hours, just trying to figure out how things work. It took some time to get accustomed to the abbreviations, [this source](https://ghc.haskell.org/trac/ghc/wiki/Commentary/Abbreviations) has some of them. I also spent some time tracking down what types things were really- for instance, I eventually found that the structure that held the bindings was a [TcIdBinder], but to know that I had to ack and dig around files that held definitions.

The structure I ended up working with was a [TcIdBinder], and a TcIdBinder is one of two things- a thing which has an "Id" or a thing that has a "Name". By looking at a method that manipulated TcIdBinders I saw there was a method call to change Id's into Name's.

So I just checked to see if Name's were shadowed.

Except that didn't work.

Some debugging later, I looked into where Name's were defined, and found the thing I really wanted to be comparing was the OccName (occurrence name). A few accessor functions later I had a working solution!

If you want to more concretely see what I mean, [here's the patch](https://phabricator.haskell.org/D2434) and you can scroll all the way down to see the code diffs.

---

# Adding a test

Okay, now to add test cases!
Following the '[adding a test](https://ghc.haskell.org/trac/ghc/wiki/Building/RunningTests/Adding)' document in GHC, I make two new files under testsuite/tests/typecheck/should_fail:

```
T12117.hs       <-   code goes in here
T12117.stderr   <-   error messages we hope to see go here
```

I also modified the file tests/typecheck/should_fail/all.T by adding the line:

```
test('T12177', normal, compile_fail, [''])
```

cool. Then I ran all the typecheck tests by going down to the typecheck directory and just running

```bash
$ make
```

I also used

```bash
$ make THREADS=4      -- or however many processors you actually have
```

and

```bash
$ make TEST=T12177
```

to only run the test that I just added. All tests passed, so I was ready to make a patch!

---

# Making a patch

I just followed the [instructions](https://ghc.haskell.org/trac/ghc/wiki/WorkingConventions/FixingBugs) about contributing to GHC. In reality making a patch took a lot more fiddling then the straightforward version below, but here's the summary:

I set up an account on [phabricator](https://phabricator.haskell.org/).

I set up arcanist, by first heading to a directory where I put various software that I need to clone, and then running:

```bash
$ git clone https://github.com/phacility/libphutil.git
$ git clone https://github.com/phacility/arcanist.git
$ export PATH="$(pwd)/arcanist/bin:$PATH"
```

&nbsp;
I committed all my changes, trying to adhere to the git commit guideline.

```bash
$ git add -A
$ git commit -m "Relevant Bindings no longer reports shadowed bindings (fixes #12176)"
```

&nbsp;

Okay, and then I just ran "arc diff" in the root directory...
And it failed, complaining about linting issues. I fixed them, readded the files to git and updated the commit, and reran arcanist.

```bash
$ git add -A
$ git commit --amend
$ arc diff
```
&nbsp;

That's it! Arcanist created [a new entry](https://phabricator.haskell.org/D2434) on Phabricator for me.
