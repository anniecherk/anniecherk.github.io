---
layout: post
title: "a love letter to workflow automation, and Just"
date: 2021-09-21
permalink: workflow-automation
categories: computers
---

In this blogpost I'll:
- talk about two concepts I find very useful: **workflow** and the **dev loop**
- argue that writing down workflows and automating them is extremely helpful for preserving and communicating knowledge about how to interact with different parts of a codebase
- explain why I automate my workflows with Just

Here is how this blogpost is organized:
- **Section 1:** Defining workflow & dev loop, and why automating workflows is useful
- **Section 2:** Just feature highlights
- **Section 3:** Two interesting situations for automating workflows
- **Section 4:** Writing down workflows is a good step towards improving them
- **Section 5:** A pragmatic look at updating and maintaining justfiles
- **Appendix:** Comparison of Just to other possible tools

# 1: Concepts: workflow & dev loop

About a year ago, I became sufficiently fed up with grinding against the mental overhead of task-switching between projects that I became motivated to look for ways I could update my development practices to make this easier. I found a few techniques that helped, but the one that's had the most outsized effect on my development process, even beyond handling this specific concern, is automating my workflows. 

**Workflow:** When I say workflow, I mean the series of commands that I run in my terminal to interact with some part of a codebase. 

Before I had the concept of a workflow, I searched backwards through my shell history, and I had messy text documents where I copy-pasted commands for later reference. A big pain point for me when switching back to working on a project after having my headspace elsewhere for weeks was the mental drag of having to reassemble all the commands that I needed to run. Automating my workflows has immensely alleviated that by collecting all those commands when they're fresh from use and crystalizing them in an executable file. Once you have such a file started, there's a low barrier to adding new commands. Capturing most (if not all) commands you run on the regular greatly lowers your cognitive load which makes developing, especially with more complicated systems, much more pleasant.

If you also have a big text file that you copy-paste commands into, you might recognize the pain of returning to that text file after some time away only to find that your commands have gone stale. Maybe:
- The way in which you need to interact with the project has changed
- You forgot to copy-paste the latest commands before stepping away from the project
- You have multiple versions of the same command and most of them don’t work anymore 

The benefit of automating your workflows in an executable file, which you then use, is that that file becomes a living artifact that you'll naturally update as your workflows change. Materializing your workflows in a file makes them a first-class part of your repository rather than an afterthought. 

What kinds of commands are useful to record and automate? In my opinion, pretty much all of them. Take, for example, [the commands recorded in the repo of the workflow automation tool Just](https://github.com/casey/just/blob/master/justfile). There are commands in there to:
- run the build
- run the tests
- run the formatter
- run the linter
- install with/without developer dependencies
- push branches to Git

The only commands I don't create workflow rules for are those which I think of as being part of the core of the tools I often use, like `git commit` or `docker ps`.

Another benefit of collecting all the commands you need to run in one file is that it is more obvious how to either improve or further automate them when they are all in one file. 

In addition to finding scripting workflows to be useful for my personal development practices, they're also useful as a way to communicate technical knowledge to others & help onboard others into a codebase. I started down this whole path of thinking about workflows because I was annoyed that task-switching between projects was a grind. I find that thinking about what I can do to help out future-me, who has inevitably forgotten some of the details, to be a pretty good proxy for what is also helpful for someone else looking to gain familiarity with the codebase.

**Dev loop:** Another big benefit I've found from the concept of a workflow is another concept: a development loop. I define a dev loop as the series of commands that I need to run after I edit my code to gain evidence that my change did or did not have the intended consequence. In many ways, writing code incrementally and debugging are both like running many back-to-back science experiments, and I've found great value in being able to rerun each experiment with a single command.

These two concepts are useful to me because they are central to how I understand codebases. It's not that I have a codebase that I Know Things about and I incidentally also run commands around it. I understand code by interacting with it in tight development loops; knowledge of how to interact with the codebase *is* the foundation upon which I build all my other knowledge. This makes it well worth my time to intentionally consider how I want to build my dev loops, and to save those decisions for myself and others to use in the future.

This isn't quite true, of course-- understanding code by reading & reasoning about it is also an integral part of understanding a codebase, but usually if I'm reading code instead of running & instrumenting it, it's because it's a pain to run. If I'm developing code for a part of a codebase, I sure as hell better be running it. If I'm working on code that is a pain to run (which happens! it might be that there's some complicated environment that needs to be configured, and/or the build / initialization process may have many steps, and/or be slow, and/or maybe I don't know what the steps are) then it's doubly valuable for me to invest the time in automating my dev loop to make running it go from painful to trivial.

Workflow & dev loops are tools for thought: I put a lot of stock in the value that concepts bring, that having a concept may allow you to think in a way that you couldn't without the concept. In this way, I've found that the concepts of workflow & debug-loops have let me think about my development practices in a way that's made me much more efficient at developing code, and happier in the process. For more on the idea that concepts are themselves tools for thought, see Michael Nielsen's [thought as a technology](http://cognitivemedium.com/tat/).


# 2. Just: a command runner

I use the tool [Just](https://github.com/casey/just) to automate my workflows. I like it, and I'll tell you why below, but there are other things you could use too. You could set up a Makefile or write a bash(/zsh/fish/rash/etc) script. Regardless of the tool you use, I think some of the most important characteristics are: having all your workflows in a single file, actually using that file in your practice, and having a uniform and documented interface to run the file. In this section, I'll talk about what I like about Just. I also briefly explain why I prefer Just over Make and a collection of shell scripts in the appendix at the very bottom of this article.

**Discoverability:** I use one justfile per project, which lives in the root directory of the project. I like having everything in a single file which is always named justfile and is always in the root directory, because the uniformity of that convention lowers the mental overhead for me looking for what the workflows are. I don't like having to look through multiple files, especially if I have to spend effort guessing at where those files might be and what they might be named. I like having all my workflows in a single file, written with rule names and maybe even comments, because that makes my workflows discoverable. Want to know what the workflows for the repo are? You can `just --list` them, or skim the justfile.

**Chaining rules together:** Just will let you encode dependencies between rules. Looking again at the justfile in the Just repo, we see that when we `just push`, that will run `just check`, which runs the lint, clippy and test rules before running some additional checks:

```bash
{% raw %}check: lint clippy test
	git diff --no-ext-diff --quiet --exit-code
	grep {{version}} CHANGELOG.md
	cargo build --features summary
	cargo +nightly generate-lockfile -Z minimal-versions
	cargo test
	git checkout Cargo.lock

push: check
	! git branch | grep '* master'
	git push github {% endraw %}
```

**Rules can take arguments:** It's frequently useful to write a rule which takes an argument and uses that argument in the body of the rule. For example, in a justfile that deals with services in a docker swarm, I have a rule for restarting a single service that looks like:

```bash
# restart a single service {% raw %}
quick-restart SERVICE:
docker service scale {{SERVICE}}=0
docker service scale {{SERVICE}}=1 {% endraw %}
```

I can call this rule with `just quick-restart mystack_server` to restart the server or `just quick-restart mystack_db` to restart the database.

**Polyglot:** In a justfile you can specify an interpreter and embed a small script in that language. I tend to use this feature in two ways. First, I'll sometimes write rules that specify the bash interpreter when I need to maintain filesystem state between the commands, like if I need to change directories:

```bash
do-something-somewhere-else: {% raw %}
	#!/bin/bash
	pushd somewhere
	./do-something
	popd {% endraw %}
```

My second use case for polyglot rules is if I have a good reason for running a small piece of code as part of my workflow. As an example, when I need to do some non-trivial file system traversal (like run a command in every sub-directory of some non-trivially nested directory structure), I much prefer to do this using python than bash. So, I might write a python module with functions for doing the specific file system traversal that I need, and in my justfile set the interpreter to be python, imported that function and executed it:

```bash
{% raw %}build-example-repos:
	#!/usr/bin/env python3
	from scripts.build_repos import find_and_run_make()
	find_and_run_make() {% endraw %}
```

# 3. Rules for a few special situations

There are two flavors of workflows that I wanted to call out in particular.

**Running commands that *must* be run together:** I work in a codebase that has state in a few different ways. Whenever I pull the latest changes in from master, I also need to pull down a docker image built by CI and update the submodules. Before I created a single justfile rule that did all three in sync, I repeatedly had strange bugs where the answer to why something weird was happening was that either the docker image or a submodule was stale. That's a very silly class of bugs to even have to think about because I can entirely avoid them by always doing all three updates in sync:

```bash
{% raw %}# Checks out master & updates to the latest state
update-master:
git checkout master && \
git pull --rebase --autostash && \
docker pull <address-to-some-image-registry> && \
git submodule update --init --recursive {% endraw %}
```

A related rule that I find useful is that I will often work on a dev branch, and once it's readyish to land, I'll catch up by grabbing the latest change off the remote master branch and merging them in:

```bash
{% raw %}# pulls the latest from master and merges it into the current branch
# also pulls the latest docker image and updates submodules
merge-master:
#!/usr/bin/env sh
cur_branch=`git rev-parse --abbrev-ref HEAD`
just update-master &&\
git checkout $cur_branch && \
git merge master
echo "\n🏃‍♀️  You've caught up to master! 🏃‍♀️🏃‍♀️" {% endraw %}
```

This rule fails every once in a while, say when I have to manually resolve a merge, but it works great almost all the time.

**Do-nothing rules:** Sometimes workflows aren't amenable to automation, but I'd like to record them anyways. For example, I'm working on a fastapi based server, and I'd like to remember that I can see and interact with the API through [the swagger UI](https://fastapi.tiangolo.com/features/). So, I make a just rule that doesn't do anything, just reminds me what to do:

```bash
{% raw %}view-api:
	@echo "check out the docs at http://localhost:8000/api/v1" {% endraw %}
```

I first heard about this idea in Dan Slimmer's [blogpost](https://blog.danslimmon.com/2019/07/15/do-nothing-scripting-the-key-to-gradual-automation/) about do-nothing scripts, and just as he argues, I find it to be a useful low-effort step towards automation when I don't immediately know how to more fully automate a workflow.

In this case, there's a better way to do this-- I could automate this by having this rule launch a new browser tab with that URL. I didn't do that though, because I don't usually launch browser tabs from the shell, so I wasn't thinking about the fact that that's possible when I wrote the rule. My goal was to record this workflow for my future reference, and writing this do-nothing rule got me most of the benefit that fully automating it would have.

There are often many possible workflows I could have to achieve the same goal, and I don't need to find the very best one. I write down the one I'm using, even if I know it's hacky, because having a hacky workflow written down is miles better than having nothing written down.

Even better, having a hacky workflow written down is the first step to having a slick workflow written down. If at some later point I have the bandwidth and motivation to improve my workflows, having them written down in one place makes it easy for me to consider how I might do something better. If I find a better way, I can just swap that rule out.

Two very good motivations for improving my workflows are making my dev loops take less time, and making them contain fewer commands that I have to remember to sequence.


# 4. Improving workflows

As I said in the intro, I think the biggest benefit of automating your workflows is lessening your cognitive load by creating this valuable artifact that encodes how to interact with parts of the code. A secondary benefit, however, is having all my workflows solidified in front of me in one place, which makes it easier for me to reason about improving them: can I make my dev loops better? 

Here are two anecdotes from my experience: 

**Making a dev loop take less time:** I was working on some code that ran as part of a docker swarm, so once I had the swarm up I had a dev loop that went: modify code, tear down stack, rebuild, stand up the stack back up, reload some state, interact with a server, and watch the logs to see the effect of my code modifications. Rinse and repeat with a rule like:

```bash
{% raw %}restart
	just down
	just rebuild
	just up {% endraw %}
```

Tearing down the stack & standing it up again made my dev loop take longer than it needed to, because it meant I was both restarting services that didn't need to be restarted and deleting system state that I then had to reload. I updated my dev loop to instead only scale down + up the service I was working on with a restart service rule, and renamed my previous restart rule to represent a clean-restart:

```bash
{% raw %}clean-restart:
	just down
	just rebuild
	just up

restart SERVICE:
	docker service scale {{SERVICE}}=0
	just rebuild
	docker service scale {{SERVICE}}=1 {% endraw %}
```

I believe I would have updated my process even if I wasn't using a justfile-- I updated my process because I was annoyed with how long it was taking. I do, however, think it was easier for me to reason about what actions I was taking because I had them written down. This is partly because I find it helpful to name the actions; my workflows have steps, and if they take a long time I can reason in terms of the steps involved and try swapping out steps for other steps.

**Making a dev loop take fewer steps:** In general, many of the examples in this blogpost are about making dev loops take fewer steps, from the example in the Just repo justfile that runs the linter and the tests as part of the rule for pushing code, to the example above which scales down a service, rebuilds, and scales the service back up in a single rule.

The fewer commands I have to remember to run in series, the more headspace I have to focus on what I'm trying to implement and the happier I am about it. 

# 5. The lifecycle of my justfiles

Here are a few thoughts on how I use justfiles in practice.

When I first start working in a codebase, I'll create a blank justfile. I'll then create rules for commands that I know how to run immediately, or as I discover them. I think so far that's pretty obvious: what I want to mention though, is how I use justfiles as my workflows or the underlying code changes. 

The point I want to make is, I work in two different mindsets: one in which I'm in codemode and I use and sometimes rip up and reassemble my workflow rules, grafting in other bits of code, and another mindset in which I carefully save, craft and clean up useful workflows as a gift to my future self. My justfile isn't pristine at all times, but it is always an extremely useful part of my development process.

As I mentioned above, I aim to run my dev loops in a single command. In practice though, that isn't always how it goes. Sometimes I have a just rule that doesn't quite work for what I need, so I'll start copy-pasting parts of the rule and running those commands directly. I'll mix those in with new commands or modifications. When I'm doing that, I'm not in organizing mode: I'm heads-down, in the flow of codemode, programming by intuition. I'll update my justfile with a new rule whenever I feel like I have the bandwidth to think about it, which might not be until many days or weeks later. Because of this, my justfiles occasionally get messy. They'll have things like multiple slightly-differently named versions of a rule, and broken or outdated rules. I'll occasionally go back and clean, cull and comment my justfiles. 

One thing that I haven't yet figured out a good solution for is how to handle workflows that are specific to uncommon interactions. For example, I might create a rule for a dev loop while working on some specific feature. I use this rule all the time while working on the feature, but it's not generally useful to me. One option is not saving these dev loop workflows at all. Otherwise, some possible places to save them include: at the bottom of the justfile, in some overflow justfile, in the body of the relevant commit, or in a related ticket or merge request. I'm currently saving them in a secondary overflow justfile, but I'm not convinced what the best option here is.


# tl;dr: takeaways

Concept: workflow. A workflow is the series of commands that I need to run to interact with some part of a codebase. 

Concept: dev loop. A development loop is the series of commands that I need to run after I edit my code to gain evidence that my change did or did not have the intended consequence.

Claim: thinking about workflows & dev loops explicitly as part of your development process is useful. They give you conceptual anchors to reason about how you interact with the code, and how you might change the ways in which you interact with your code to make your life more pleasant. 

Action: automating your workflows is useful for a myriad of reasons. It:
- is a concrete way to think about workflows
- lowers the cognitive effort of development
- lowers the barrier to improving your workflows
- reifies the knowledge of how to interact with the codebase for both you and others 




# Appendix: Tool comparison

### Just vs. Make: 
I prefer Just over Make for recording workflows because:
- in Make you'd have to make your rules [.PHONY](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html); it's clear from the sentiment of the word "phony" that Make wasn't designed to run commands in this way
- rules can take arguments. Makefile rules can also use arguments but [it's weird](https://stackoverflow.com/questions/2214575/passing-arguments-to-make-run)
- I can write inline python (or whatever language) with the polyglot feature
- I don't have to be Very Concerned about [using tabs](https://stackoverflow.com/questions/2131213/can-you-make-valid-makefiles-without-tab-characters)

Further, I think there's a bit of conceptual difference between a workflow and rules for running a build system, though it's a blurred distinction. I want to look in a file that I know contains workflows when I need a map for navigating codebase interactions, and I want to look in a file that contains build automation when I need to configure or change something about the build.

### Just vs. shell scripts: 
I prefer Just over a [collection of] shell scripts because:
- all the workflows are collected in a single file
- I can write inline python (or whatever language) with the polyglot feature
- Justfiles have a syntax for explicitly denoting dependencies between rules, which I think is easier to understand at a glance than the shell script equivalent of reading through a function to see which other functions it calls. That's maybe splitting hairs though.

Similar to the sentiment in the comparison to Make, I think a justfile conveys a different intent than a shell script. A repository may have many scripts, possibly containing many functions: some of those scripts / functions may be intended to be invoked directly, and some may be there as helper scripts / functions. Which is which, for what purpose, and how to invoke them may not be immediately apparent. Conversely, the purpose of a justfile is consistent across codebases.

**Aside: Just vs. CI runner scripts:** 
While they’re not the right choice for recording all your workflows, I find that CI runner scripts will often have at least some of the workflows you may be using encoded in them. If you've ever had the experience of looking through a CI runner config and learning something useful about your build or deploy process, using a justfile is like that-- but for all your interactions. Ideally CI (and all the developers on the project) would use the justfile directly, so that the workflows themselves are tested and pinned to the state / version of the repository.

### Tool comparison conclusion: 
Just is my preferred tool for workflow automation because it is a tool built specifically for the job of encoding workflows. It has some nice features like: 
- making the workflows easily discoverable
- chaining workflows together
- passing arguments into workflows
- letting you write other languages inline

At the end of the day, though, I hope your takeaway from this article is that automating workflows is worthwhile; let's not have a spaces-vs-tab debate about how to do it. I like using task-specific tools: I've got a grapefruit knife and a garlic press in my kitchen. If you like to mince your garlic & section your grapefruit with the only knife in your knife block, you do you.

