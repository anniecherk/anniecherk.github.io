---
layout: post
title: "project management preflight checklist"
date: 2024-01-27
permalink: project-management-preflight-checklist
categories: computers
---



I think of "project management" as everything related to committing that a particular scope of work will be done by a particular date, and then increasing the odds that it actually gets done as promised.

I have strong feelings about project management because working on a project that is going poorly sucks. Having an opinion as early in the project as possible about what is (and is not) valuable to do, and how, and when, is, in my experience, the only avenue for minimizing probable future suffering.


![me working on a project that is going poorly](/images/project_on_fire.png)


It's easier said than done. Projects of every kind, scale and media go off the rails all the time. My favorite fiasco is the Sydney Opera House which was [a decade late and 1300% over budget](https://en.wikipedia.org/wiki/Sydney_Opera_House#Completion_and_cost). 

In my opinion, the main culprits are: 

- there are so many different possible complications. It's impossible to account for everything that can go wrong and how likely / impactful each complication is

- social / political pressure to make overly-optimistic estimates 

- failure to adjust gracefully to changing realities

The goal isn't to have every project go without a hitch, but instead to improve the odds that a project goes as well as possible given the resources and context of a given situation. This includes both building the best thing given the circumstances, and leaving everyone involved without a sour taste in their mouth.

Below are some principles that guide my thinking about project management and questions I find useful to ask along the way. 

I continue to find new and exciting ways to be wrong about how long and difficult a project will be, but I think the rate at which I'm being surprised is decreasing. I'm just hoping that by the time I build a national opera house, the timeline will slip by less than a decade.


# Principles

  
<p></p>

### Do not build the wrong thing

<p></p>


A week of coding can save you an hour of planning. 

Q: How do you know whether you are building the right thing?

A: Experiment and iterate

- best approach: get an MVP in front of users
    - this also lets you fail fast when your hypothesis turns out to be wrong
- next best: find out from stakeholders what concerns they want to address. ie, product research or direction from customers
    - not necessarily reliable; pay more attention to what the underlying concerns are rather than the specifics
    - do not assume you know or understand what they want; keep asking questions
- worst approach: form your own opinions about what is useful and build that in one go
    - if you want accurate hypotheses, you must look for anti-evidence

![Construction cats thinking about what they should do](/images/thinking_cats.png)

### Build incrementally in priority order
- always work on the most important thing
- at every point imagine that you got blind-sided by another project and no longer have time to work on this current project. Another way to say this is, imagine that instead of the timeline you thought you had, you now only have until Tuesday. Have you built the most core important thing thus far?
    - don't hang up the paintings until you've finished the dry-wall
- this is another flavor of "do not build the wrong thing"

### Front-load risk by prioritizing uncertain tasks first
- if the plan hinges on a step you're unsure will work, prove that part out first
    - otherwise, if it turns out not to work, all the effort leading up to it may need to be thrown out as you pivot to another approach
- if you're not sure if the plan has steps that might not work, figure that out ASAP

### Set expectations & re-negotiate as things change
- not "if things change" but "as things change" because they will
- if there's a non-negotiable scope, engineers must be able to define the timeline; if there's a non-negotiable deadline then engineers must be able to define the scope. ("you tell me what, I tell you when; you tell me when, I tell you what")
- every situation is negotiable; situations that are presented as non-negotiable are ones where someone is choosing not to negotiate
- you will get new requirements after the project starts. 
    - it's not a question of "if" you can add a new feature to the execution plan, but whether it is worth removing something else from the plan to make time
    - time is bounded and non-expandable. You cannot add more features by working more. Hoping that there will be enough extra time is not a strategy.
- if a deadline slips, involve stakeholders in renegotiations
    - this builds buy-in on a new plan given the reality of the situation
- keep detailed notes


### Plan only when it's worth reducing uncertainty
- if you can tolerate higher uncertainty, executing is a better use of time than planning
- it's worth investing in planning if any of these things are true:
    - real consequences to missing the original delivery date
    - social / political pressure to commit to a tight deadline
    - limited ability to renegotiate after initial timeline commitment


### If you need a solid estimate, break down the project into smaller tasks
- given a solid estimate and acceptable uncertainty, don't sweat the details
    - you can't predict the future
    - instead of trying to predict the future choose a comfortable cushion
    - the amount of uncertainty that is acceptable depends on the situation & on the consequences of failing to meet the deadline 

![A cat questioning its life decisions](/images/after-hours-cat.png)

### Optimize for maintainability

The hardest part of software engineering is maintaining the system in the future in the face of shifting use cases and unforeseen business directions. That will be more effort than writing the code. Make decisions guided by maintainability rather than ease of implementation.
- we can't predict the future, so bias towards flexibility to allow pivoting in the future
- prefer data representations that generalize best
- build a system that allows for the data representation to change in the future

### Choose to take on tech debt intentionally
- some (even many) code quality complaints are not worth addressing
- categories of tech debt that must be addressed immediately for a long-lived project:
    - it must be clear what the code is doing; code should be verbose
        - test cases are useful as documentation
        - abbreviated variable names are not acceptable. For those following golang's half-century old ideology of sml vrbl names, may I recommend the marvelous modern tooling that we've developed since the typewriter days, such as tab completion
    - code must have tests. Untested code is hard to change in the future because you don't have confidence that your future changes won't break something. _run your tests often / don't let them go stale / rejoice when they pass & rejoice when they fail_
- if you find yourself repeating sequences of steps, [automate them](https://anniecherkaev.com/workflow-automation)
    - especially if they are error prone


### Beware of overpromising & underdelivering
- it sounds obvious, but it's surprisingly tempting to be optimistic
- no one likes surprises. Reliable estimates lead to better outcomes than tight deadlines


![A cat wondering what we are doing and why](/images/what-are-we-doing-and-why.png)

<p></p>
  
<p></p>

# Pre-flight checklist


  
<p></p>


### Scoping:
- what are we doing and why?
	- who are the stakeholders?
	- what concerns do they hold?
		- what is the root of the result they want? 
		- sometimes specific requests obscure a simpler way to achieve the same level of functionality / satisfaction
	- how will we evaluate whether what we build fulfills the intended goal
	- what if we don't do this project?
	- if we do this, what will we not do instead? Is this more important than whatever is in the "wont do" pile?
- what must be in the MVP?
	- this is the MVP according to whom?
	- for each item:
		- what if we didn't include it?
		- is there a simpler version that we can include?
- what's missing? Things to consider:
	- how will we know the implementation is correct?
	- what operational metrics do we care about & how will we get them?
	- what will we do when we find a bug in production? 
	- what are the known error conditions?
	- how will we launch / rollout the project?
- for each item in MVP:
	- how much uncertainty is there? How possible are time-consuming surprises?
		- is it worth building any proof of concepts?
	- break down large or uncertain tasks into finer tasks
		- trade-off between effort spent up-front vs taking it as it comes; how much it is worth putting effort into this depends on how important it is to give a reliable estimate
	- any interactions with other systems to consider?

### Sanity-checking the estimate:
- how does this compare to other projects (ie a lot smaller, the same, a bit bigger)?
	- does that comparison make sense?
- effort audits of past projects
	- what was surprising? why?


![A cat reading the "code 4 cats" manual](/images/code-4-cats.png)

### Execution:
- who is working on what?
	- are they around?
		- planned vacation or leave?
	- do they have other commitments? Is it possible / likely that those go into crunch time?
		- if they have other project work in parallel, how painful will task-switching be?
	- do they want to be working on this project? Will it be challenging & interesting to them?
- what are the dependencies?
	- what are the deepest dependencies? Untangle these first
- what might not work? Where do we have uncertainty about how to implement?
	- de-risk these first
- what new work have we discovered while executing?
	- how important is it that we do it immediately?

### Scope creep:
- does this need to be fixed / built / modified right now?
	- useful to be intentional in what tech debt to take out and what to pay down


### Remember that shit happens:
- technical complications arise
	- someone is stuck
	- a large flaw or bug is revealed
		- if resolution is not cheap & obvious, explore possible remedies before committing to an approach
- external pressures
	- people get pre-empted, pulled to other efforts
		- requests related to compliance & security
		- urgent requests from company priorities
		- on-call requires support, ie large or complicated incidents
	- code freeze, holidays & deployment / merge schedules may create complicated constraints 
	- chaos from above: reorgs, layoffs, abrupt changes in direction and priorities

<p></p>
  
<p></p>

## Summary

To recap, the goal of project management is about how to do the best with what you've got, both in terms of what you build and doing right by everyone involved.

Summarizing the ideas of how to do that:
- be as selective as possible about what to build
- get feedback from customers / users early & often
- keep everyone apprised and ready to re-prioritize and pivot as the ground shifts
- optimistic timelines do not do anyone any favors, but it's hard not to give optimistic timelines even knowing this


<p></p>
  
<p></p>
  
<p></p>
____________

![Three cats at peace](/images/good-job-cats.png)

<p></p>
  
<p></p>

## Appendix

All the images here are from midjourney 🤖❤️

#### thank yous

I don't know how to attribute all the ideas above to the people who shaped my thinking, but I can at least thank some of them: most emphatically Rohan Ranade & Elise Jiang, as well as Zack Thomsen-Gray, Scott Moore, Tristan Ravitch, Avery Pennarun's blog on [planning](https://apenwarr.ca/log/20230605) and on [tech debt](https://apenwarr.ca/log/20230605), and Martin Nally's [talk on designing quality APIs](https://www.youtube.com/watch?v=P0a7PwRNLVU) 
