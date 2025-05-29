---
layout: post
title: "workflows for ai coding"
date: 2025-05-28
permalink: on-the-brink-ai-coding
categories: computers
---


workflows for ai coding, 05/25

its bananas out there.

six months ago ai coding tools were take-them-or-leave-them, and now they have foundationally changed the way i work. and things are different by the week.

here is how i think it's worth approaching ai coding as of late may 2025:

0. use the tools
1. use the tools, better
2. improve the tools as you use them
3. improve your workflows for using the tools

there are lots of great guides on steps 0-2 of how to work with ai coding tools, like [this one](https://www.anthropic.com/engineering/claude-code-best-practices). generally i'm not trying to provide many notes on how to use these tools both because i think they'll change very quickly, and because there are already many good guides. but i provide a few ideas, briefly, below because i can't help myself.

mostly though, i want to talk about a few thoughts about step 3, *workflows* for interacting with ai tooling. 

i have an embarrassing secret which is that i was/am a productivity nerd. this means i have spent a lot of time assuaging my fears about how i'm not doing enough things by thinking about how to do more things, rather than actually doing them. in retrospect, this was a bad strategy because the fear of not doing enough things was a proxy for a deeper and scarier fear of not being enough, which, coincidentally, is not something you can fix with really good todo lists.

anyway, because of this, i have a lot of thoughts and opinions about todo lists, and how to take notes, and about attention and focus. a thing that interests me about ai tooling is how strongly ideas about how to organize personal workflows and notes and knowledge systems translate to workflows for working with ai coding tools.

the reason i'm writing this is there are two concrete things related to all this that i want to talk about:

1. prompting tools these days generates 1-3 minute gaps in my workflows while it spits out code or text. there are folks in the industry already reporting letting their coding models run unattended for hours at a time, presumably that will make it's way down to all of us in the pretty near future. but, either way, with a gap of a minute to a few hours while working on a task, the way we work is going to have to change drastically. waiting is really hard. task switching is really expensive. i talk about some more thoughts and things that may help, at least for my brain.

2. ai tooling generates a lot of code, which means i have to manage and organize a lot of code now. i thought i was good at git, but then i learned better workflows, and now i think these are foundationally important for ai coding. i talk about these workflows.

the other reason i'm writing this is it feels to me like we are at a tipping point for what it means to do software engineering. that feels cool and exciting and important and weird to be in the middle of it. i want to write down so i remember what it feels like to be standing here, looking forward and looking back.


----------------------------------------------------------------------------------------------------------

# background: a few thoughts on using ai tools well

### 0. use the tools:

- the tools work pretty well out of the box.  just try them.
	- try claude code
	- try roo plugin for vscode

### 1. use the tools better:
	- use better prompts, ie:
		- "don't make assumptions; ask me clarifying questions"
		- "when I do X I see Y issue. list possible reasons why this might be the case, consult with me before implementing."
		- "for solution Z, how will we know if that fixes the bug? write out a plan to validate and consult with me before you start."
	- modes: use the right mode for the task
		- in roo: "ask", "debug", "code", "architect"
	- use the right model for the task
		- larger context models do better at "ask" and "architecture" tasks
	- understand what you can use the tools to do
		- some examples beyond "code", "debug" and "explain":
			- explain the history of a piece of code by looking at git
			- refactor a codebase by first adding tests to constrain the existing functionality
			- automate gathering data for a report
			- perform operational tasks: monitor deploys, ingest alerts and triage
			- project management
			- code review, incl explaining what a commit does & flagging complicated issues like async thread coordination

### 2. collaborate with the tools to improve themselves, via:
	- save prompts, and tune them
		- [claude.md](https://docs.anthropic.com/en/docs/claude-code/memory), .roo/01_git_instructions.md
	- turn crib sheets into saved prompts
		- you know the random notes you have in some txt file somewhere on how to do something? that is the smell of a thing you should automate
- architect first, then implement
	- "we need to implement feature X. Make a plan for how to do that, write it down and ask me to review"
	- work with external systems and libraries:
		- mcp servers, use them and create them
			- an mcp server is just a programmatic description of an API
		- summary notes
			- grab repo X and take notes on how feature Y works, write them down
	- sessions
		- have it take notes on what it needs to do, and what it has done so you can split the work across context windows
- *go slower to go faster*
	- i need this on a sticky note on my desk
	- taking the time to make improvements to the tools pays dividends, and i think we're in a moment where the dividends are huge. improving the tools becomes a non-optional step.


### 3. improve your workflows for using the tools
	- see below
	- be prepared to change, repeatedly
	- once things feel stable, coordinate with your team. [work together, be strong](https://qupqugiaq.com/)

----------------------------------------------------------------------------------------------------------

# working with models is not ergonomic for my attention span, and what to do about 

currently my ai coding tasks run for a small handful of minutes. as this gets better, maybe they'll run for an hour or several hours. now and in the future, this is a very different way of working than iterating on code in a flow state. 

so what should i do with the downtime gaps?

presumably the answer is, and will increasingly need to be, work on multiple things at once.

the problem is that, at least in the naive implementation, it is really painful to task switch at that granularity. at least for the way my brain works. trying to swap between tasks whenever i have a minute of downtime feels like pulse width modulation.

<todo: image>


### pair programming, by myself

there is a blogpost that i have wanted to write for five years, called pair programming by myself. it's about how i take notes. i have written several drafts that i've never published because when i read them they always sound too silly-- the one sentence summary is i have found it absurdly helpful to take notes that are extremely overly detailed. i have a lot of feelings about detailed notes but i also feel ridiculous waxing on about it. so let me tell you the short version now:

in mid-2020, i suddenly stopped getting much work done, in a way that's probably familiar to you. hello, pandemic. i had a to-do list of things i probably should have been doing, but my brain was laser focused on the latest scrap of info-- was everyone i loved going to get sick and die? were our supply chains going to collapse? what was happening out there?

and eventually in all this i had a project deadline come up, and thought i really really should get some work done. it was just impossibly hard to focus. and eventually i hacked it for myself with a ridiculously detailed todo list. it had things like:
- open the terminal
- cd into the project directory
- start jupyter notebook
- open the browser
- navigate to the notebook
- run the notebook, make sure it still compiles
- etc

it worked. i got my brain unstuck. i have a lot of feelings about how well this works for me, and i think this doesn't generalize to all brains and certainly not to all tasks. my notes aren't usually literally as detailed as the example above, but to give a sense of scale i write about two pages a day as i work. i still find it very helpful to separate out the planning from the flow state of doing. 

*it's not about the notes, it's about the way the notes enable my thinking.* my notes enable me to extend my working memory and become better organized and oriented. i think they also let me stop mentally re-cataloging all the things i need to keep track of, which lets me focus.

that sure sounds a lot like how running these models in architect / planning mode first gets much better results.

a common division of roles in [pair programming](https://en.wikipedia.org/wiki/Pair_programming) is the driver and the navigator. the driver is the person at the keyboard. the other person is the navigator; they're responsible for having a bigger picture plan in mind and discovering the higher level strategic concerns. 

i think of the process of making detailed plans as i work as pair programming by myself.

### pair programming, stuck in navigator mode

i have the feeling that as our ai coding models get increasingly better at driving, we're going to be increasingly in navigator mode.

(ai coding models can and do also navigate: they work better if you ask them to plan and align on their plans before they jump into coding. so really we'll be spending more time in some meta-navigation mode).

so i think then the question should be: what contexts make it easier to navigate?

### pitch of a solution: consolidate navigating into a focus block

i suspect that an answer, for me and my brain, is to consolidate navigating.

specifically what im trying is writing out the steps of several tasks for the day at the start of the day. tjis lets me spend more time in breaking-up-work mode. and then when i switch to detail-oriented mode, i can stay there.

it doesn't work all the time or for all tasks, but its better for my brain than one task at a time


#### aside on brains

when i say that this works for my brain but may not work for yours, i do really believe that. brains are different! it blew my mind to learn that people are pretty evenly split between having an internal monologue and not having an internal monologue.


----------------------------------------------------------------------------------------------------------
# git skills

if you asked me a year ago if i was good at git, i'd say of course i'm good at git. and then one of my coworkers showed me much better workflows, and my life was much improved. 

however, i think these skills are pretty foundational for working with ai coding tools
- ai coding tools generate a lot of code quickly
- you have to manage all that code effectively
- or else something will break and you wont know when where or why
	- keep track of where the nearest exit is, keeping in mind it may be behind you

here are the specific recommendations, or skip ahead to the code below:

### branches are free
here is how i used to code:
- write code
- finish writing code
- pick and choose pieces to put into a PR

this is how i write code now:
- create a WIP branch
- have the ai commit every time it finishes a task
- sometimes i add my own commits
	- these are mostly to remind myself that the state is known good, or known bad
- finish writing code
- cut a new PR branch, from the WIP branch
- blow away all the commits from the PR branch !
	- this seemed REALLY SCARY because i was sure i'd fuck up and delete files i cared about
	- but all the files i care about are safe&sound on the WIP branch!
- carefully split into commits, using rebase
- carefully split code review iterations across commits, using rebase

but also with a bit of this thrown in:
- WIP branch
- PR branch
- bug fixes on the PR branch
- cut a NEW PR branch
- some refactoring on the new PR branch
- cut a real PR branch for real this time
- use that to open a PR

branches are free! 

to manage a proliferation of branches, i have an alias to get recent branches so i remember what i have in front of me. i provide it below but also you can just ask your robot friend to write it for you.

### commits are free

i have told the robot how commits [should be structured](https://cbea.ms/git-commit/). every time it finishes a task, it commits what happened in that task

i blow away these commits but it's nice to have these in case it starts down a path that ends up being rotten. then you have context on how "known good" your various checkpoints are.

### commands

```
git checkout -b ac/my-feature-wip
# lots of commits, mostly auto-gen'd, commit ALL the changes even temporary files

git checkout -b ac/my-feature-pr
git fetch
git reset origin/main # blows the commits away!

alias recent="git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short)' | head -n 10"

# make some commits, probably using github desktop for careful line selection
git rebase -i origin/master
# tweak commit messages

# open a PR
# make some code review motivated change
git add <my change>
git commit -m "squash me after commit blah"
git rebase -i origin/master
# rearrange the new commit to be in the right spot, then squash it
```

----------------------------------------------------------------------------------------------------------
# conclusion

when ai coding tools started coming out and we saw headlines like "will coding jobs go away" i tried the tools and thought that, like most headlines, they were silly. 

today i no longer think that. the definition of software engineering is changing week to week right now. 

i think it's a prudent time to pay attention and understand the new tools available to us, learn them, and imagine how we can leverage them. 

and understand that that will also be completely different in another six months.

it feels to me like we're in the middle of a laddering up on the tech tree, and it happens to be in the tech that is the craft that i practice. that feels extremely special and exciting. i'm reminded of the beginner's mind

the two things i'm trying to remember right now, in my attempt to see the ocean from my place on the wave are:
- go slower to go faster
- change is coming, change is happening, be prepared to change again and again and again

----------------------------------------------------------------------------------------------------------

#### aside

i didn't use ai to write this blogpost. i get the heebiegeebies from aesthetic homogeneity (looking at you, cooking blogs), and as there are more blogposts that have that corpopolished gleam to them, the more that speaks to my sense of dont tell me what to do. i have a lot of empathy for folks in creative fields who lament ai automating the fun, meaningful, and quality-inducing parts of their jobs. that sucks, i'm not stoked for that future. i want to make things that look like what i want them to look like, and i want you to make things that look like what you want them to look like
