---
layout: post
title: "workflows for ai coding"
date: 2025-05-28
permalink: the-times-they-are-ai-changing
categories: computers
---

its bananas out there.

six months ago ai coding tools were take-them-or-leave-them. now they have foundationally changed my day to day work. and things are changing by the week.

### approaching ai coding in mid 2025

here is how i think it's worth approaching ai coding as of late may 2025:

0. use the tools
1. use the tools, better
2. improve the tools as you use them
3. improve your workflows for using the tools

there are lots of great guides on steps 0-2 of how to work with ai coding tools, like [this one](https://www.anthropic.com/engineering/claude-code-best-practices). generally i'm not trying to provide many notes on how to use these tools both because they'll change very quickly, and because there are already many good guides. but, because i can't help myself, i included a few brief thoughts in an appendix.

### ai coding workflows

mostly though, i want to talk about a few thoughts about step 3, *workflows* for interacting with ai tooling. 

i have an embarrassing secret which is that i was/am a productivity nerd. this means i have spent a lot of time assuaging my fears about how i'm not doing enough things by thinking about how to do more things, rather than actually doing them. in retrospect, this was a bad strategy because the fear of not doing enough things was a proxy for a deeper and scarier fear of not being enough, which, coincidentally, is not something you can fix with really good todo lists.

anyway, because of this, i have a lot of thoughts and opinions about todo lists, and how to take notes, and about attention and focus. a thing that interests me about ai tooling is how strongly ideas about how to organize personal workflows and notes and knowledge systems translate to workflows for working with ai coding tools.


### tl;dr

there are 3 things that i want to talk about:
```
human attention
```

prompting tools these days generate 1-10 minute gaps in my workflows while they spit out code or text. which sucks.

even as this improves to hours, the way we work is going to have to change drastically. 

waiting is really hard. task switching is really expensive. i talk about some more thoughts and things that may help, at least for my brain.

```
git workflows for managing lots of code changes
````

ai tooling generates a lot of code, which means we have to manage a lot of code changes now.

i thought i knew git pretty well, but after learning some better workflows, i now think that having better git workflows is going to be crucial for managing huge volumes of code changes. i talk about these workflows.

```
my feelings about the change we're experiencing
```

the other reason i'm writing this is it feels to me like we are standing on the precipice between an old world and a new world. software engineering, as a craft, is changing by the moment. it feels cool and exciting and significant and weird to be in the middle of it. i want to write down so i remember what it feels like to be standing here, looking forward and looking back.


----------------------------------------------------------------------------------------------------------

# human attention span 

currently my ai coding tasks run for a small handful of minutes. in the near future, they will run for an hour or several hours, and that will be a worthwhile improvement. now and in the future, this is a very different way of working than iterating on code in a flow state. 

so what should we do with the downtime gaps?

presumably the answer is, and will increasingly need to be, work on multiple things at once.

the problem is that, at least in the naive implementation, it is really painful to task switch at that granularity. at least for the way my brain works. trying to swap between tasks whenever i have a minute of downtime feels like the worst kind of pulse width modulation.
```
wait what was i doing?        _  _   _    __     ____
task b:            _ ___    _         _                        _________    
task a:   __|  |__|     |__|  |____|   |__   |__     |__|__|__ 
time --> 
```

### pair programming, by myself

there is a blogpost that i have wanted to write for five years, called pair programming by myself. it's about how i take notes. i have written several drafts that i've never published because when i read them they always sound too silly-- the one sentence summary is i have found it absurdly helpful to take notes that are extremely overly detailed. i have a lot of feelings about detailed notes but i also feel ridiculous waxing on about it. so let me tell you the short version now:

in mid-2020, i suddenly stopped getting much work done, in a way that's probably familiar to you. hello, pandemic. i had a todo list of things i probably should have been doing, but my brain was laser focused on the info about the world-- were we all going to die? were our supply chains going to collapse? *aside:* i love connie willis, but [the doomsday book](https://en.wikipedia.org/wiki/Domesday_Book) was a not a great choice of bookclub book in April 2020.

and eventually i had a project deadline come up, and thought i really really should get at least, you know, *some* work done. it was just impossibly hard to focus. and eventually i hacked it for myself with a ridiculously detailed todo list. it had things like:
```
- open the terminal
- cd into the project directory
- start jupyter notebook
- open the browser
- navigate to the notebook
- run the notebook, make sure it still compiles
- etc
```

it worked. i got my brain unstuck. it reminds me of uma thurman in kill bill "just wiggle your big toe". i have a lot of feelings about how well this works for me, and i think this doesn't generalize to all brains and certainly not to all tasks. my notes aren't usually literally as detailed as the example above, but to give a sense of scale i write about two pages a day as i work. i still find it very helpful to separate out the planning from the flow state of doing. 

**it's not about the notes, it's about the way the notes enable my thinking.** my notes enable me to extend my working memory and become better organized and oriented. i think they also let me stop mentally re-cataloging all the things i need to keep track of, which lets me focus. i also have a lot of feeling about this, [and highly recommend reading michael nielsen's "thought as technology"](https://cognitivemedium.com/tat/).


### pair programming, stuck in navigator mode

a common division of roles in [pair programming](https://en.wikipedia.org/wiki/Pair_programming) is the driver and the navigator. the driver is the person at the keyboard. the other person is the navigator; they're responsible for having a bigger picture plan in mind and discovering the higher level strategic concerns. 

i think of the process of making detailed plans as i work as **pair programming by myself**.

i have the feeling that as our ai coding models get increasingly better at driving, we're going to be **increasingly in navigator mode.**

*aside:* ai coding models can and do also navigate: they work better if you ask them to plan and align on their plans before they jump into coding. so really we'll be spending more time in some meta-navigation mode.

so then the question should be: **what contexts make it easier to navigate?**

### pitch: consolidate navigating into a focus block

i suspect that an answer, for me and my brain, is to consolidate navigating.

specifically what im trying is writing out the steps of several tasks for the day at the start of the day. this lets me spend more time in breaking-up-work mode. and then when i switch to detail-oriented mode, i can stay there.

it doesn't work all the time or for all tasks, but its better for my brain than trying to constantly interrupt myself to switch between two tasks

```
wait what was i doing?               _     _          _      __
task b:            ________    __     __     __    __     __       __
task a:   _________        |__|  |__|   |__ |  |__|  | __|  |   __| 
time --> 
```

### pitch: longer focus blocks

i think the **real** permenant solution will be longer focus blocks: the models have to run longer without baby-sitting. there are reports of folks already doing this! this just isn't the case for most things that i do today. 

any opportunity to improve how long a model can run uninterrupted is very worth investing in

#### aside on brains

when i say that this works for my brain but may not work for yours, i do really believe that. brains are different! it blew my mind to learn that people are pretty evenly split between having, and not having, an internal monologue. i have non-stop words in my head all the time, which seems correlated with why it's so helpful to think in writing via detailed notes.


----------------------------------------------------------------------------------------------------------
# git skills

if you asked me a year ago if i was good at git, i'd say of course i'm good at git. and then one of my coworkers showed me much better workflows, and my life was much improved. 

i now think these skills are pretty foundational for working with ai coding tools
- ai coding tools generate a lot of code changes quickly
- you have to manage all that code effectively
- or else something will break and you wont know when where or why
	- keep track of where the nearest exit is, keeping in mind it may be behind you

here are the specific recommendations, or skip ahead to the code below:

### branches are free
here is how i used to code:
```
- write code
- finish writing code
- pick and choose pieces to put into a PR
```

this is how i write code now:
```
- create a WIP branch
- have the ai commit every time it finishes a task
- sometimes i add my own commits
	- these are mostly to remind myself that the state is known good, 
	  or known bad
- finish writing code
- cut a new PR branch, from the WIP branch
- blow away all the commits from the PR branch !
	- this seemed REALLY SCARY because i was sure i'd fuck up and 
	  delete files i cared about
	- but all the files i care about are safe&sound on the WIP branch!
- carefully split into commits, using rebase
- carefully split code review iterations across commits, using rebase
```

but also with a bit of this thrown in:
```
- WIP branch
- PR branch
- bug fixes on the PR branch
- cut a NEW PR branch
- some refactoring on the new PR branch
- cut a real PR branch for real this time
- use that to open a PR
```

branches are free! 

to manage a proliferation of branches, i have an alias to get recent branches so i remember what i have in front of me. i provide it below but also you can just ask your robot friend to write it for you.

### commits are free

i have told the robot how commits [should be structured](https://cbea.ms/git-commit/). every time it finishes a task, it commits what happened in that task

i blow away these commits but it's nice to have these in case it starts down a path that ends up being rotten. then you have context on how "known good" your various checkpoints are.

### commands

```
git checkout -b ac/my-feature-wip
# lots of commits, mostly auto-gen'd
# commit ALL the changes even temporary files

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

#### what am i missing?
do you have sick git workflows that i should know about? hit me up pls, inquiring minds want to know

----------------------------------------------------------------------------------------------------------
# conclusion

it feels to me like we're in the middle of a laddering up on the tech tree, and it happens to be in the tech that is the craft that i practice. that feels extremely special and exciting. it's a time to review our assumptions about what is possible.

it's a prudent time to pay attention and understand the new tools available to us, learn them, and imagine how we can leverage them. it's about unlocking projects and possibilities that were too expensive in effort before. 

the two things i'm trying to remember right now, in my attempt to see the ocean from my place on the wave are:
- go slower to go faster
- change is coming, change is happening, be prepared to change again and again and again

----------------------------------------------------------------------------------------------------------
<br/>

#### aside

i didn't use ai to write this blogpost. i get the heebiegeebies from aesthetic homogeneity (looking at you, cooking blogs), and as there are more blogposts that have that corpopolished gleam to them, the more that speaks to my sense of dont tell me what to do. i have a lot of empathy for folks in creative fields who lament ai automating the fun, meaningful, and quality-inducing parts of their jobs. that sucks, i'm not stoked for that future. i want to make things that look like what i want them to look like, and i want you to make things that look like what you want them to look like

<br/>

----------------------------------------------------------------------------------------------------------

# appendix: using the tools [better]

[recap] here's a map of how to approach a relationship with ai tooling:

0. use the tools
1. use the tools, better
2. improve the tools as you use them
3. improve your workflows for using the tools

some brief notes:

### 0. use the tools:
    - the tools work pretty well out of the box.  just try them.
	    - try claude code
	    - try roo plugin for vscode 

### 1. use the tools better:
	- use better prompts, ie:
		- "don't make assumptions; ask me clarifying questions"
		- "when I do X I see Y issue. list possible reasons why 
		   this might be the case, consult with me before implementing."
		- "for solution Z, how will we know if that fixes the bug? 
		write out a plan to validate and consult with me before you start."
	- modes: use the right mode for the task
		- in roo: "ask", "debug", "code", "architect"
	- use the right model for the task
		- larger context models do better at "ask" and "architecture" tasks
	- understand what you can use the tools to do
		- some examples beyond "code", "debug" and "explain":
			- explain the history of a piece of code by looking at git
			- refactor a codebase by first adding tests to constrain 
			  the existing functionality
			- automate gathering data for a report
			- operational tasks: monitor deploys, ingest alerts and triage
			- project management
			- code review, incl explaining what a commit does, and
			  flagging complicated issues like async thread coordination

### 2. collaborate with the tools to improve themselves, via:
	- save prompts, and tune them
		- [claude.md](https://docs.anthropic.com/en/docs/claude-code/memory), 
		   .roo/01_git_instructions.md
	- turn crib sheets into saved prompts
		- you know the random notes you have in some txt file somewhere 
		on how to do something? smells like a thing you should automate
	- architect first, then implement
		- "we need to implement feature X. Make a plan for how to do that, 
		write it down and ask me to review"
		- work with external systems and libraries via mcp servers & summaries
		- sessions
			- have it take notes on what it needs to do, and what it has 
			done so you can split the work across context windows
	- **go slower to go faster**
		- i need this on a sticky note on my desk
		- taking the time to make improvements to the tools pays dividends, 
		and i think we're in a moment where the dividends are huge. 
		improving the tools will quickly become a non-optional step


### 3. improve your workflows for using the tools
	- <the rest of the article>
	- be prepared to change, repeatedly
	- once things feel stable, coordinate with your team. [work together, be strong](https://qupqugiaq.com/)