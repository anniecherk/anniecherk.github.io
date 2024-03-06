---
layout: post
title: "blue team project scoping"
date: 2024-03-05
permalink: blue-team-scoping
categories: computers
---

![lets go blue team](/images/red_team_blue_team_2.png)

# Red team vs Blue team

In cybersecurity, red team is the attackers trying to break into a system, and blue team is the defenders trying to keep them out. Red team has the easier role; red team needs only one vulnerability to get in. Blue team needs to discover all possible vulnerabilities before red team and patch them.*

In practice, of course, blue team can't do that. You can't prove the absence of a weakness so it's a Sisyphean task. In practice blue team has two tools: limiting impact, and shoring up defenses by patching the biggest holes in the defense. 

As an example, a ransomware attack is where an attacker gets onto a computer system, encrypts all the files, and offers to unencrypt them in exchange for a ransom. (Sending ransoms is, of course, one of three core uses for bitcoin, along with [money laundering](https://www.bloomberg.com/opinion/articles/2023-01-19/ftx-is-planning-a-comeback) and [securities fraud](https://en.wikipedia.org/wiki/Sam_Bankman-Fried).) As the blue team, you can fortify your defenses by keeping the OS up to date and using a strong password, but better yet you can mitigate the impact of having your files encrypted by keeping a backup of your device. If you have an up-to-date backup then there's no reason for you to pay to get your files back, as you never lost your files in the first place. Goodbye, [red team blues](https://us.macmillan.com/books/9781250865847/redteamblues)!

![nothing to see here, red team](/images/red_team_blue_team_1.png)

# Blue team development

As someone trying to develop and operate a stable software system, you are the blue team, and bugs are the red team. 

Limiting impact looks like having a deploy setup where you frequently ship small code changes and pushbutton rollbacks. That doesn't prevent bugs from getting in, but when they do, you can roll back to the latest stable version quickly & easily thereby limiting the impact of the bug.† 

Patching the largest holes in the defense looks like shoring up against bugs. You cannot prove that you have written a system that does not have bugs.‡ You can write test cases for all the scenarios you can think of, but for all your tests you still won't know you don't have bugs. As is true for any blue team, you need to invest proportionally such that the most obvious holes in your defenses are patched.

If you have a way to limit the impact, patching the holes matters a lot less for system stability. It is wise to prioritize limiting impact over shoring up defenses.

These tools aren't either/or. If you have your files backed up, you still don't want to use `password` as your password. If you have a pushbutton rollback, it's still better not to use it. It's good to have the option, but you probably have better things to do with your time. The difference is that if you have a way to limit impact, it changes from being a question of system stability to a question of development velocity. Development velocity is a nice thing to contemplate once nothing is actively on fire.‖

So of course it's useful to have tests, but less to keep the bugs out and more to not constantly trip over your own feet. If you can rollback to a stable version at any point, failure is more about getting in the way of your ability to deliver than it is about impacting system stability.

Okay cool story, but "have a way to rollback changes to give you system stability" isn't a newsflash. Having frequent, small deploys has been an industry standard for a long while. The real reason I'm interested in this is because what I want to talk about is blue team project scoping. Blue team development was actually also an analogy, in the way that the blue team itself was an analogy. Sorry but it's analogies all the way down.

![no red team blues over here](/images/red_team_blue_team_3.png)

# Blue team project scoping


Most software projects aren't about literally setting up CI/CD systems, so yeah, hope you have a CI/CD system to limit the impact of bugs but that's not what I'm thinking about with my time as a software dev. 

Many projects do however have to handle concerns about how to roll a system out safely to production, and then how to safely operate and iterate on it.

Managing scope is hard, even when you know managing scope is hard. There are lots of worthwhile things to be doing, and it can be challenging to figure out which ones to prioritize and in what order. Having thought-tools for prioritization can help.

**Blue team scoping is thinking about project scope, especially with respect to rollout and operations, and prioritizing limiting impact of failure over shoring up the defenses.** If you can:

- be sure you know when a failure has occurred (ie be confident in your alerting), and
- handle failure gracefully (ie quickly, and preferably automatically)

then you have limited the impact of a potential bug. Having thorough test is also useful, but not as a mechanism to shore up defenses against failure, but rather as a mechanism to support quick & confident iteration.§ As in the section above, even if you *can* safely fail, recovering from failure is still not the best use of your time. 

Given a long list of possible tasks to consider, it can be hard to order them because there are many seemingly good reasons to argue for different orderings & scopes. Blue team project scoping is a thought-tool for prioritizing work that limits the impact of bugs over other possible work to establish development bedrock.


![red team is blue team is red team is blue team](/images/red_team_blue_team_4.png)

## Footnotes

\* Okay, easy is an understatement. See [google project zero](https://googleprojectzero.blogspot.com/) for accounts of contemporary zero days. 

† Okay, you can, but most systems aren't [formally verified](https://en.wikipedia.org/wiki/Formal_verification) and ones that are have guarantees with narrow operating conditions. From the perspective of an engineer and not a researcher here in 2024, practically you can't guarantee that a real system does not have bugs.

‡ Okay, that's not quite true, having pushbutton rollbacks doesn't protect you from other sources of failure, notably infrastructure & data issues.

‖ Okay, development speed is _also_ important. Working slowly can be an existential risk for several reasons. As a developer, trading stability for development speed is an unfun tradeoff that's worth making sometimes given context. But blah blah blah I'm just going to sit here thinking sunny thoughts.

§ Okay, also as a mechanism to shore up defenses against failure, and lots of other reasons too. Tests are great documentation, especially when viewed as a spec about edge-cases system behavior. There's a lot of value in preserving hard-won knowledge in bugfixes; my favorite thoughts on this are [this article on legacy code](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/).