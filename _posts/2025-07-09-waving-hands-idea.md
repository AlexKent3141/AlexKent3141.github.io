---
layout: post
title: "Project Idea: Waving Hands"
date: 2025-07-09 12:00:00 +0000
categories: []
tags: [game, project, idea]
---
<div markdown="1" class="sidenote-box">
# Work in progress
I'll update this post as I flesh out my plan.
</div>

For this post I wanted to jot down some ideas for a project that's been on my mind for a while.

Waving Hands is a Pen-and-Paper game that was created by Richard Bartle. I'm not sure exactly when it was created, but the [Rules Document](http://www.gamecabinet.com/rules/WavingHands.html) was last edited in 1994, so before then.

In this game you play as a (powerful) wizard who casts spells by making gestures with your hands. You can make gestures with both hands, so you have two streams of spells going on at any given time. The opponent is also casting spells, and you and your opponent's gestures are resolved simultaneously on each turn, making for some interesting interactions.

I'm not going to present all of the rules now (refer to the document linked above) but the gestures are:
* Wiggled fingers (F)
* Proferred palm (P)
* Snap (S)
* Wave (W)
* Digit pointing (D)

There's a large roster of spells of the form:

| Gestures | Spell
| C-D-P-W | Dispel magic
| W-F-P | Cause light wounds
| W-F-P-S-F-W | Summon giant

As you can see in this subset, there's some overlap between spells. If you choose gestures efficiently you can gain access to more powerful spells. If you have to change plan, and target a different spell, that's going to cause you to waste turns.

# Why is this cool?
I love the concept of two people gesturing at each other wildly while dealing magical damage. For me it evokes a Harry Potter wandless-magic vibe.

<div markdown="1" class="sidenote-box">
#### Lore
I used to work at a company called Ultraleap and one of the main projects (and the bit I worked on) was hand-tracking using custom camera hardware.

Ultraleap's tech is super high-quality, but comes with a price tag. It would be fun to explore some open source options.
</div>

I think I could use existing hand-tracking technology to really bring this game to life.

# Development ideas

#### 1. Core implementation
Implement the rules of the game: we need to keep track of the state and be able to generate and apply gestures.

The output of this stage of development should be a library which exposes an API for mutating the game state.

At this point the game is not playable, but we should have unit tests to gain confidence that the implementation is correct (including the example game given in the rules document).

#### 2. MVP: playable command line game
Terminal UI which allows us to play the game either human vs human or human vs random play.

This will consume the core implementation library produced in the last step.

The key UI element we need is a tracker for the spells we're lining up.

#### ?. Search algorithm to provide an interesting opponent
I think it will be pretty tricky to make a compelling computer opponent for humans. As a game, Waving Hands doesn't have a very high branching factor (gestures for both hands give 25 possibilities, then you potentially have targeting options) but it does have:
* Hard to evaluate, partially complete, spells
* Random elements (the confusion spell, maybe others?)
* Simultaneous moves

Assuming I can get something to work, I think we could make some interesting custom opponents by limiting the spell repertoire of the computer opponent.

<div markdown="1" class="sidenote-box">
#### Note
A few years ago I would have probably referred to this search algorithm as "AI". Today "AI" is a very overloaded marketing term, so I've made a conscious decision to avoid using it for this project.
</div>

#### ?. Graphical UI
This is a long way off, but I could have a lot of fun rendering spell effects.

#### ?. Hand-tracking
Being able to fully control the game with your hands would be the ultimate goal. To make this work I foresee the need to have some kind of "commit" gesture to apply your actions.

# Algorithms & tech choices
#### Programming language
I like the idea of tackling the Core library and terminal UI in a system programming language. For me `C` is the obvious choice (it probably existed when the game was invented which is kind of nice). I've also done some recent experiments with `Odin` lang, so that's a possibility.

Certainly the Core library should present a `C` interface to make it easy to consume.

#### Keeping track of active spell gestures
This seems like it could be done nicely with a modified [Trie](https://en.wikipedia.org/wiki/Trie) structure. In Waving Hands we wouldn't have terminal states because spells can flow into one another, but my intuition is that we could extend the trie structure to be a graph.

#### Search algorithm
[MCTS](https://en.wikipedia.org/wiki/Monte_Carlo_tree_search) could probably be adapted to do the job (it side-steps the evaluation problems and deals with randomness well) though simultaneous moves are still awkward to deal with.

A good first pass might be to use straight Monte Carlo simulations.

#### Hand-tracking
It looks like there are some good open source options for this, including Google's [mediapipe](https://github.com/google-ai-edge/mediapipe) project.

