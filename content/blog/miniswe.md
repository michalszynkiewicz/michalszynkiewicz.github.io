---
title: "miniswe: A Lightweight Coding Agent for Local LLMs"
description: "A lightweight, Rust coding agent built for small local LLMs"
date: 2026-06-29
---

A while back, I wondered how we could help a small local model work well with code.
Thankfully, nowadays top models make it easy to put ideas into code.

That's how [`miniswe`](https://github.com/michalszynkiewicz/miniswe) was born. 
Written in Rust because I have limited trust in the NPM ecosystem after recent supply chain attacks. 


## Core features
The tool is built around structured planning, context compaction, and a focused toolset.

### Planning
The core idea of miniswe is explicit planning. Given a task, the model is first asked to take a look around, and create a plan for the work. Then, it is expected to mark plan steps done. For some steps, the model can decide to require the project to be compilable when the step is done.
The last step has to produce compilable code.

### Done gate
In addition to the plan, miniswe comes with a done gate. When analyzing the task, the model is asked to determine how to check if the task is done. This is used as a gate before claiming success on the task.

### Context handling
LLMs work best when used as a conversation. It allows them to take into account things that happened before. This brings one major problem. The context grows over time.
With small models, on user's hardware, it's especially visible. Instead of 1M tokens of context, we have tens of thousands of tokens. This makes handling context a non-trivial task. How do we preserve what matters but free up enough not to blow up the context?
Miniswe uses the model to perform smart compaction of the old messages, and keeps the new messages raw. 
I plan to make a larger post on context handling next.

### Repository map
A small model has limited context. If it were to analyze the project on its own, it could easily bloat the context. `aider` - a well-established competitive tool - provides tree-sitter based repo map with PageRank to solve this problem. `miniswe` reuses this approach.

### Fast feedback for the model
To keep the model on track, and stop it from going into loops of deteriorating changes, miniswe provides tree-sitter + LSP feedback on edits.

## The results
I benchmark the tool by asking it to add an additional flag to the `miniswe` project itself. This means pushing the value of the flag through a few layers of method calls, and updating multiple call sites. To balance speed and capability, I'm using gemma4 26B MoE, running on RTX 3090. And it works!

## Give it a try
If you'd like to give `miniswe` a try, go to [GitHub releases](https://github.com/michalszynkiewicz/miniswe/releases) and download one for your system.

Pull requests and issues with feedback are more than welcome!