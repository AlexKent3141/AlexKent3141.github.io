---
layout: post
title: "Workflow: Using The Command Line Effectively"
date: 2025-08-15 12:00:00 +0000
categories: []
tags: [workflow, cli, bash]
---

My engineering workflow is significantly different to that of my current professional software engineer peers.

Every other engineer in my team, and in parallel teams, uses an IDE (typically Visual Studio Code or CLion for my C++ colleagues, and exclusively Jetbrains IntelliJ for Java engineers) as their primary development environment.

I'm not typically known for being non-conformist, so why do I stand out in this situation?

The answer is that I find a pure command line workflow to be the most effective way to work.

I've been a software engineer for 15 years and I've slowly built up my repertoire of core command line tools. I'm still discovering ways to make things better, so expect a longer list later!

## Core Tools

# Reverse-search
OK, this one is more of a feature of the terminal than a standalone tool, but it's so important that I have to mention it. Even if the command line isn't your primary development environment it is critical that you use reverse-search to speed up your interactions with it.

Scenario: you're struggling to remember that `curl` command you figured out yesterday which made an RPC call to the server. How did it go again? Fortunately reverse-search has your back:

```
ctrl-r -> you've entered reverse-search mode

curl -X POST ... -> suddenly the command is completed using the terminal's history!

curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "give_me_burger", "params": ["extra_gherkins"], "id": 1}' http://mcd.food
```

Amazing! Of course as a professional I document every API call thoroughly, but this saves me a *lot* of time. It's very flexible: you can also search for the middle of a string or choose to look further back in the history if the first match isn't what you want.

# Vim
You need to pick a good editor and learn it well. I've been a `vim` user for many years and I'm at the point where navigation and common flows are muscle memory.

There are lots of popular alternatives to `vim`, from the classic GNU Emacs to modern options like Zed. I personally haven't explored them too deeply.

One extra bonus of using `vim` is that it's normally present by default on Linux boxes, so it's very useful to know the basics if you frequently SSH into remote machines.

# Tmux
I'm slightly embarrassed to admit it, but for quite a while I would open tons of terminal windows for different purposes and spend a painfully long amount of time Alt-Tabbing my way between them to find the right one. This was very awkward and probably slower than using an IDE for the same task.

The solution was to use the terminal multiplexer `tmux`.

With `tmux` the flow becomes:
1. Open a terminal
2. Run `tmux`
3. Create as many windows/panes as you want and have convenient switching

To give you an idea of how this might look, my current "optimal" setup is to open one `tmux` window for each repo/project I'm interested in and then split each window into up to 4 panes (which are essentially sub-terminals within a window) with one in each quadrant.

Just like with editors, there are multiple choices available for terminal multiplexers. I've personally used a newer one called `terminator` with similar results, but `tmux` is my current favourite.

# ctags
Including `ctags` in my workflow was the most recent significant improvement. One thing I didn't have access to that IDEs are very good at is functionality like "go to definition" for functions and variables, but `ctags` gives me a simple and fast way to achieve something very close to this.

1. In the repo root run `ctags -R .` to build an index file for all symbols in the repo.
2. Now in `vim` move the cursor to the symbol you want to locate and use the shortcut `ctrl-]`.

This will take you to the best guess declaration location `ctags` has for the symbol. It can't deal with ambiguity very well (e.g. function overloads) so you can instead try `g ctrl-]` to get a list of possible matches to pick from.

So far I've been very impressed by `ctags` and I'm still learning how to make best use of it. I'm sure there are integrations in other editors but so far I've only used the `vim` plugin.

## But why?
Hopefully some of the above tools have piqued your interest and you want to try them in your own workflows.

I honestly believe that for my use-cases (primarily C++ and Python development) the command line approach is faster than using a graphical IDE: you can simply achieve a higher APM using the keyboard exclusively than you can when you add a mouse into the mix.

I'm sure there are cases when this is not the most efficient way though. The Java developers I know swear by IntelliJ, so I think it must be a very good fit for them.

## LLM Tools
I'm actively trying to integrate command line LLM tools into my workflow and I'm seeing benefits from using GitHub Copilot and Google's Gemini CLI. There's a lot of development going on in this space: while I wouldn't add any LLM tools to my Core Tools list yet I expect some will be on there in the future.
