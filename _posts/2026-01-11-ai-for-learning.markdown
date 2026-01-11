---
layout: post
title: "choosing learning over autopilot"
date: 2026-01-11
permalink: choosing-learning-over-autopilot
categories: computers
---

I use ai coding tools a lot. I love them, I'm all-in on ai tools. They unlock doors that let me do things in a way that I cannot do with my human hands alone.

But they also scare me.

As I see it, they offer me two paths:

The glittering vision is they let me build systems in the way that the version of me who is a better engineer would build them. Experimentation, iteration and communication have become cheaper. This enables me to learn by doing at a speed that was prohibitive before. I can make better decisions about what and how to build because I can try out a version and learn where some of the sharp edges are in practice instead of guessing. I can also quickly loop in others for feedback and context. All of this leads to building a better version of the system than I would have otherwise.

The cursed vision is I am lazy, and I build systems of ai slop that I do not understand. There's a lot of ink spilled about perils and pains of ai slop, especially working on a team that has to maintain the resulting code. 

The thing that scares me the most about this actually though, is an existential fear that I won't learn anything if I work in the "lazy" way. There is no substitute for experiential learning, and it accumulates over time. There are things that are very hard for me to do today, and I will feel sad if all of those things feel equally hard in a year, two years, five years. I am motivated by an emotional response to problems I find interesting, and I like problems that have to do with computers. I am afraid of drowning that desire by substituting engaging a problem with semi-conscious drifting on autopilot.

And part of why this is scary to me is that even if my goal is to be principled, to learn, to engage, to satisfy my curiosity with understanding, it is really easy for me to coast with an llm and not notice. There are times when I am tired and I am distracted and I have a thing that I need to get done at work. I just want it done, because then I have another thing I need to do. There are a lot of reasons to be lazy.

So I think the crux here is about __experiential learning__:
- ai tools make it so much easier to learn by doing, which can lead to much better results
- but it's also possible to use them take a shortcut and get away without learning
    - I deeply believe that the shortcut is a trap
    - I also believe it is harder than it seems to notice and be honest about when I'm doing this

And so, I've been thinking about guidelines & guardrails-- how do I approach my work to escape the curse, such that llms are __a tool for understanding, rather than a replacement for thinking__? 

Here's my current working model:
1. use ai-tooling to learn, in loops
1. ai-generated code is cheap and not precious; throw it away and start over several times
1. be very opinionated about how to break down a problem 
1. "textbook" commits & PRs
1. write my final docs / pr descriptions / comments with my human hands

The rest of the blog post is a deeper look at these topics, in a way that I hope is pretty concrete and grounded.

# but first, let me make this more concrete

#### Things I now get to care less about:
- the mechanics of figuring out how things are hooked together
- the mechanics of translating pseudocode into code
- figuring out what the actual code looks like

The times I'm using ai tools to disengage a problem are the times I'm doing the things in the "things I get to care about less" category to go fast and getting away with skipping doing the things in the "things I should still care about" category.

#### Things I cared about before and should still care about:
- deciding which libraries are used
- how the code is organized: files & function signatures
- leaving comments that explain _why_ something is set up in a way if there's complication behind it
- leaving docs explaining how things work
- understanding when I need to learn something more thoroughly to get unblocked

#### Things I now get to care about that were expensive before:
- more deeply understanding how a system works
- adding better observability like nicely structured outputs for debugging
- running more experiments

The times when I'm using ai tools to enhance my learning and understanding I'm doing the things in the "things I should still care about" category and not taking advantage of the "things I now get to care about" category.

I will caveat that the amount of effort and care is contextual to the problem and context. I believe that having a team move in overly conservative ways carries engineering risk. It also sucks to be on a very conservative team, I think fear can lead to micromanagement and fighting ghosts. The real key, I think, is understanding how much effort is worth investing and where.

I like to work on problems somewhere in the middle of the "how correct does this have to be" spectrum and so that's where my intuition is tuned to. I don't need things clean down to the bits, but how the system is built matters so care is worth the investment.

# workflow

Here is a sketch of a workflow I've been using for working on medium-sized problems.

#### Get into the problem: this is the stage to go fast, be messy, learn and get oriented

1. Research & document what I want to build
    1. I collab with the ai to dump background context and plans into a markdown file
    1. A format that I've been using: 
        1. What is the problem we're solving?
        1. How does it work today?
        1. How will this change be implemented?
    1. The doc at this stage can be rough
1. Build a prototype
    1. The prototype can be ai slop
    1. Bias towards seeing things run & interacting with them
1. Throw everything away. Start fresh, clean slate
    1. It will take so much longer to fix the PoC than to build it correctly next time, now that I know what all the pieces are and how they should relate.

#### Formulate a solution: this is the stage to figure out what the correct structure should be

1. Research & document based on what I know from the prototype
    1. Read code, docs and readmes with my human eyes
    1. Think carefully about the requirements & what causes complication in the code. Are those hard or flexible (or imagined!) requirements?
1. Design what I want to build, take 2
  1. Now would be a good time to communicate externally if that's appropriate for the scope. Write one-pager for anyone who might want to provide input. 
1. Given any feedback, design the solution one more time, and this time polish it. Think carefully & question everything. Now is the time to use my brain.
    1. Important: what are the APIs? How is the code organized?
    1. Important: what libraries already exist that we can use?
    1. Important: what is the iterative implementation order so that the code is modular & easy to review?
1. Implement a skeleton, see how it smells
1. Use this to compile a final draft of how to implement this iterative
1. Commit the skeleton + the final implementation document

#### Implement the solution: this is the stage where I work on the final code, because I now have confidence that I know what I need to build and how I need to build it

1. Cut a new branch & have the ai tooling implement all the code based on the final spec
1. If it's not a lot of code / it's very modular, review it and commit each logical piece into its own commit / PR
1. If it is a lot of code, review it, and commit it as a reference implementation
    1. Then, rollback to the skeleton branch, and cut a fresh branch for the first logic piece that will be its own commit / PR
    1. Have the ai implement just that part, possibly guided by any ideas from seeing the full implementation
1. For each commit, I will review the code & I'll have the ai review the code
1. I must write my own commit messages with descriptive trailers


Now let me briefly break out the guidelines I mentioned in the intro and how they relate to this workflow.


# learning in loops

There are a lot of ways to learn what to build and how to build it, including:
- Understanding the system, surrounding systems and existing work
- Understanding the problem, the requirements & existing work in the space
- Understanding relationships between components, intended use-cases and control flows
- Understanding implementation details, including tradeoffs and what MVP looks like
- Understanding how to exercise, observe and interact with the implementation

I'll understand each area in a different amount of detail at different times. I'm thinking of it as learning "in loops" because I find that ai tooling lets me quickly switch between breadth and depth in various aspects in an iterative way. I find that I "understand" the problem and the solution in increasing depth and detail several times before I build it, and that leads to a much better output.

One of the glittering things about ai tooling is that it's faster than building systems by hand. I maintain that even with these added layers of learning before implementing, it's still faster than what I could do before while giving me a richer understanding and a better result.

I also want to expand on the theme that it is harder than it seems to notice when I'm being lazy. 

I think there is a key here in knowing when to break out of gathering knowledge via ai summary, and reading the original sources myself. I have two recent experiences top-of-mind informing this:

In the first experience, I was debugging a mysterious issue related to some file-related resource exhaustion for certain workloads and a coworker was debugging the same issue in parallel. We both used ai tools to figure out what cli tools we had to investigate and to build a mental model of how the resource in question was supposed to work. I got stuck after getting output that seemed contradictory, and wasn't sure how to proceed because it didn't fit the mental model I had built with the help of ai. My coworker got to a similar spot and then took a step out of the ai tooling to go read the docs about the resource with their human eyes. That led them to understand that the ai summary wasn't accurate: it had missed some details that explained the confusing situation we were seeing. This example really sticks out in my memory: I thought I was being principled rather than lazy by building my mental model of what was supposed to be happening, but I had gotten mired in building that mental model second-hand instead of reading the docs myself.

In the second experience, I was working on a problem related to integrating with a system that had a documented interface. I had the ai read & summarize the interface and then got into the problem in a way similar to the first step of the workflow I described above. I was using that to formulate an idea of what the solution should be. Then I paused to repeat the research loop but with more care: I read the interface with my human eyes-- and found the ai summary was wrong! It wasn't a big deal and I could shift my plans, but I was glad to have learned to pause and take care in validating the details of my mental model.


# ai-generated code is throw-away code

I had a coworker describe working with ai coding tools like working on a sculpture. When they asked it to reposition the arm, it would accidentally bump the nose out of alignment. 

The way I'm thinking about it now, it's more like: instead of building a sculpture, I'm asking it to build me a series of sculptures.

The first one is rough-hewn and wonky, but lets me understand the shape of what I'm doing.
The next one or two are just armatures.
The next one might be a mostly functional sculpture on the latest armature; this lets me understand the shape of what I'm doing with much higher precision.

And then finally, I'll ask for a sculpture, using the blessed armature, except we'll build it one part at a time. When we're done with a part, we'll seal it so we can't bump it out of alignment.

A year ago, I wasn't sure if it was better to try to fix an early draft of ai generated code to be better, or to throw it out. Now I feel strongly that ai-generated code is not precious, and not worth the effort to fix it. It takes a long time to write code by hand. If you know what the code needs to do and have that clearly documented in detail, it takes no time at all for the ai to flesh out the code. So throw away all the earlier versions, and focus on getting the armature correct.
 
Making things is all about processes and doing the right thing at the right time. If you throw a bowl and that bowl is off-center, it is a nightmare to try to make it look centered with trimming. If you want a centered bowl the time at which you have the opportunity to center it is when you are throwing. Same here, if you want code that is modular and well structured, th~e time to do that is before you have the ai implement the logic. 

## "textbook" commits and PRs

It's much easier to review code that has been written in a way where a feature is broken up into an iteration of commits and PRs. This was true before ai tooling, and is true now.

The difference is that, even if I aspired to this before, sometimes (often?) I'd get lost in the flow in a way that made it a huge pain, lift and effort to break apart into "textbook" commits after the fact.

I believe that especially if I work in the way I've been describing here, ai code is cheap. This makes it much easier/cheaper for me to break apart my work into ways that are easy to commit and review. 

My other guilty hesitation is I never liked git merge conflicts and rebasing branches. It was confusing and had the scary potential of losing work. Now, ai tooling is very good at rebasing branches, so it's much less scary and pretty much no effort.

I also think that small, clean PRs are an external forcing function to working in a way that builds my understanding rather than lets me take shortcuts: if I generate 2.5k lines of ai slop, it will be a nightmare to break that into PRs.


##  i am very opinionated about how to break down a problem

I'm very opinionated in breaking down problems in two ways:
- how to structure the implementation (files, functions, libraries)
- how to implement iteratively to make clean commits and PRs

The only way to achieve small, modular, reviewable PRs is to be very opinionated about what to implement and in what order. 

Unless you're writing a literal prototype that will be thrown away (and you're confident it will actually be thrown away), the most expensive part about building a system is the engineering effort that will go into maintaining it. It is, therefore, very worth-while to be opinionated about how to structure the code. I find that the ai can do an okay job at throwing code out there, but I can come up with a much better division and structure by using my human brain.

A time I got burned by not thinking about libraries & how to break down a problem was when I was trying to fix noisy errors due to a client chatting with a system that had some network blips. I asked an ai model to add rate limiting to the existing http client in a codebase, which it did by implementing exponential backoff itself. This isn't a very good solution, surely we don't need to do that ourselves. I didn't think this one through, and was glad a coworker with their brain on caught it in code review.


## i write my docs / pr descriptions / comments with my human hands

Writing can serve a few distinct purposes: one is communication, and distinct from that, one is as [a method to facilitate thinking](https://notes.andymatuschak.org/Evergreen_notes). The act of writing forces me to organize and refine my thoughts.

This is a clear smell-test for me: I must be able to write documents that explain how and why something is implemented. If I can't, then that's a clear sign that I don't actually understand it; I have skipped writing as a method of thinking.

On the communication side of things, I find that the docs or READMEs that ai tooling generates often capture things that aren't useful. I often don't agree with their intuition; I find that if I take the effort to use my brain I produce documents that I believe are more relevant. 

This isn't to say that I don't use ai tooling to write documents: I'll often have ai dump information into markdown files as I'm working. I'll often have ai tooling nicely format things like diagrams or tables. Sometimes I'll have ai tooling take a pass at a document. I'll often hand a document to ai tooling and ask it to validate whether everything I wrote is accurate based on the implementation.

But I do believe that if I hold myself to the standard that I write docs, commit messages, etc with my hands, I both produce higher quality documentation and force myself to be honest about understanding what I'm describing.


# Conclusion

In conclusion, I find that ai coding tools give me a glittering path to understand better by doing, and using that understanding to build better systems. I also, however, think there is a curse of using these systems in a way that skips the "build the understanding" part, and that pitfall is subtler than it may seem.

I care deeply about, and think it will be important in the long-run, to leverage these tools for learning and engaging. I've outlined the ways I'm thinking about how to do best do this and avoid the curse:
1. use ai-tooling to learn, in loops
1. ai-generated code is cheap and not precious; throw it away and start over several times
1. be very opinionated about how to break down a problem 
1. "textbook" commits & PRs
1. write my final docs / pr descriptions / comments with my human hands


