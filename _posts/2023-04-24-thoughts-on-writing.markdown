---
layout: post
title: "writing for lazy readers"
date: 2023-04-24
permalink: thoughts-on-technical-writing
categories: computers
---

My goal when writing a technical document is to create something that is easy to read. 

I am a lazy reader. I skim things. Sometimes I read a blogpost and I can't recall much except the topic sentence. I try to set up my technical writing so that when lazy readers like me read my writing they recall at least the main idea I'm trying to convey.

As much as I love whimsy, weird fonts, cookbooks with gilt edges and poetry books with unusual bindings, I like my technical writing to be the opposite-- concise and literal. Maybe one day I will be a better writer and then I will write design documents in the form of [dialogs between cute animals](https://thelittletyper.com/) but for now I'm sticking to short sentences & bulleted lists.

Below is a summary of how I think about technical writing. At the bottom there's a list of my favorite articles on technical writing.

---------------------------------------------------------------------------------------------------------------

# Structure

### Lazy reading

Here is the formula I learned for reading research papers:

- read the abstract + intro
- read all the section headers
- read the conclusion
- look at the graphs
- optional: maybe read the paper

I try to write all kinds of documents assuming that my audience is following a similar strategy. 

### Writing for lazy readers

Here is the analogous formula that I follow when writing:

- write a concise intro explaining why the audience should read the rest of the content
- use headings and/or bolding to convey the main points at a glance
- start each section with a roadmap of what that section will contain
- conclude each section with a concise summary
- include visuals when possible

This format helps the reader identify and pinpoint which sections they want to read.

### Start with an elevator pitch

Here is the formula that I follow for an elevator pitch:

**Sentence 1:** what is the problem you are trying to solve \
**Sentence 2:** what is the consequence of solving this problem (why should the reader care?) \
**Sentence 3:** a one sentence summary of your approach


If I have thought deeply about something, it is interesting to me. The reader, however, probably won't find the details interesting apriori. Therefore, if I start by explaining *why* they should care, they will be more likely to hang in there with me for the rest of the thoughts.

When you explain why the reader should care, you are telling a story. Human brains really like stories, and in particular we are motivated by the stories that we tell ourselves about the consequences of our actions.

In a technical document, you are probably telling a story. When you argue in favor or against an approach, you are telling a story. When you argue to prioritize or deprioritize a piece of work, you are telling a story. 

By starting with an elevator pitch, you are offering your reader a role in the story you are telling.

### The right level of detail for the expected audience

My ideal outcome as a reader is being told the information I need to know, only at the time that I need to know it. In practice this may mean being told the same concepts several times at increasing levels of details, like iteratively zooming in on a map.

Multi-level summaries are a tool to let readers navigate in approximately this way. Another tool is to push off details into appendices & links / references / footnotes instead of including them in-line.

---------------------------------------------------------------------------------------------------------------

# Format

Here is a list of the formatting standards I find most useful as a reader. See any of the other links on my blog for examples :)

### Format lists as lists rather than in-line

ie, don't do this: 

> The rest of this document outlines possible approaches to consider in painting the bikeshed which include doing nothing, painting it in rainbow stripes or making it glow in the dark.

do this instead:

> The rest of this document outlines possible approaches to consider in painting the bikeshed which include:
> - doing nothing
> - painting it in rainbow stripes or
> - making it glow in the dark

### Semi-structured logging; not just for machines

There may be some meta-data you want to provide in your text such as who wrote it and when.

Instead of including this kind of info in-line, extract it into an explicit structured header, ie:

> author: anniecherk \
> date: April 24, 2023 \
> status: bit-rotting

### Short sentences, short paragraphs

ie, don't do this:

> This formula can feel hard to follow. If I have thought deeply about some technical thing, I want to tell you about the details of the thing I've thought deeply about because it is interesting to me. You as the audience, however, probably don't care apriori about the thing I want to tell you about. 

do this instead:

> If I have thought deeply about something, it is interesting to me. The reader, however, probably won't find the details interesting apriori.


## Editing

The above example is verbatim from my first and final draft respectively. My first goal is to get the ideas out in whatever form, and only later make the ideas easier to understand through all the words.

When I edit my writing, my goals are:

- refine the text so that the main ideas I want to convey are clear and front & center
- rewrite to have more concise sentences, paragraphs & ideas

If the main ideas are not lit up in neon lights, I look for ways to make them more obvious including:

- bulleted lists at the beginning and/or end that lay out the take-aways
- using bolding or a different text color to highlight key ideas

---------------------------------------------------------------------------------------------------------------

# Sources / Recommended Reading
 

- Matt Might's advice on how to write emails; also applicable to other types of writing: 
[https://matt.might.net/articles/how-to-email/](https://matt.might.net/articles/how-to-email/)

- Julian Shapiro's writing handbook, especially the section on re-writing: 
[https://www.julian.com/guide/write/rewriting](https://www.julian.com/guide/write/rewriting)

- Simon Payton Jones on how to write a great research paper; also applicable to other types of writing:
[https://www.microsoft.com/en-us/research/academic-program/write-great-research-paper/](https://www.microsoft.com/en-us/research/academic-program/write-great-research-paper/)

- Dan Luu on why you may consider learning to type faster (tl;dr, typing faster lets you iterate through ideas faster):
[https://danluu.com/productivity-velocity/](https://danluu.com/productivity-velocity/)

- Andy Matuschak on how to write notes that are useful to yourself in the future: 
[https://notes.andymatuschak.org/Evergreen_notes](https://notes.andymatuschak.org/Evergreen_notes)

- I can't find sources now for who taught me how to read research papers & give elevator talks, but sending my thanks into the universe :thanks:


