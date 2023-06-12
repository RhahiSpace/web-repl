---
title: "start here"
date: 2023-06-11
mathjax: false
draft: false
series:
tags: []
---

## For the readers

Hello Kerbonauts and strangers on the Internet. In this blog, I play Kerbal Space Program with [RP-1](https://github.com/KSP-RO/RP-1/wiki/Introduction-and-Overview) mod, and write about developing rocket control software using [KRPC](https://krpc.github.io/krpc/) mod.

### How to read this blog

- If you want a natural flow of reading, take a look at "Dream"
- If you are interested in specific problems or topics, dive right into "Build", "Apply", or "Learn".
- If you want to see videos and mission results, see "Fly".
- If you want to contact me, open an issue in [GitHub](https://github.com/RhahiSpace/web-repl). For private messages, you may prefer to email `discuss@rhahi.space`. You can also chat with `Rhahi` in the [KRPC discord server](https://discord.gg/c8c36UM).

### What happened to the previous posts?

I wiped out previous posts. They will remain in git history, so if you are returning for a specific post, look at the git history. Some of them will return as they fit with the story outlined in *Dream*.

### What programming language will I use and why?

I will be writing mostly in Julia, because it is easy to write, is fast, and has access to various scientific software packages. Everything will be open source.

### The house rule

In RP-1, there are two types of flights: test flight and real flight. In test flight, or "Simulation Mode", we can build, test, and revert launches without any risk to the program. In real flight, in-game time and budget, as well as the lives of pilots (if any) are on the line. I have two rules to give some sense of gravity into the game play, keeping myself more cautious.

1. Test flights are free.
1. No reverts are allowed in real flights, with exceptions if the missions are disrupted by:
    - 1: bugs in the game itself.
    - 2: irrecoverable KRPC error.
    - 3: computer or KSP crash
    - 4: if it was otherwise necessary to do so.

## For the author

Recently, RP-1 released a version 2 of the mod. The already excellent mod looks even better now, I am starting anew in this version, and I have rebooted this blog. Here are my thoughts on how this blog should be written and read.

### What kind of blog should REPL be?

I am not going to write a journal of playing RP-1 itself. [Encyclopedia Kerbonatica](https://pap1723.github.io/RitS-RP1-MSA/index.html) is a very cool project, but I want to write a blog that I would enjoy reading. I don't think I would enjoy reading detailed mission history, performance data, and successes or failures of rockets made in a video game.

I want this blog to be a place where I can write about what I am interested in, explain what I learned, and showcase what came out of it -- hence the "Dream/Learn/Apply/Fly" structure (see also: [Build-Fly-Dream](https://www.youtube.com/watch?v=23pkcrHggtw))

### How should I write REPL?

I think it will be nice to provide a natural narrative to read the entire blog from start to end, whenever possible. The main driving force of the blog is that I am playing Kerbal Space Program with RP-1 mod. Then, maybe I can write in an order like:

1. I play the game. The game will throw me interesting challenges, or maybe I may set up a challenge myself.
2. Post "Dream": Based on the challenge, I write a "mission description" that outlines what I am going to write, learn, and do.
3. Post "Build/Learn/Apply": I write about ideas, experience, or something I learned and want to share.
    - Build is about rocket/spacecraft design.
    - Learn is about reading textbooks or documentations.
    - Apply is about programming.
4. Post "Fly": I write a mission report, with a video and relevant data and the pointer to the source code.

The linear story that connects 2, 3, 4 becomes a "series".

I think this structure can work.

### Style and writing guide

- Prefer easy-to-read sentences over "professional" writing.
- Assume various levels of readers and provide useful links.
- Prefer "I" when addressing my thoughts or what I have done.
- Use "we" when I am talking to the virtual reader and we are thinking together.
