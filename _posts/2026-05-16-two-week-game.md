---
layout: post
title: I Made a Game in 14 Days. Here’s What I Learned
description: What Brittle Blocks taught me about development cycles and player feedback.
tags: [game dev, godot]
categories: [Godot]
date: 2026-05-16 13:57 +0200
---

Been a while since I posted here. Don’t worry, I haven’t forgotten about my blog!\\
Today’s post is about my latest game, Brittle Blocks, and how it helped me let go of perfectionism and the fear of showing my work to the world (just a little). Before I dive into the development and lessons learned, I need to establish some context first.

## The Plight of Perfectionism

Over the last 10 years, I’ve started various game projects but rarely finished any of them. That’s for a number of reasons, including being busy with school/university, not believing in myself, and depression. Some of them have been discussed in my [Dark Souls post](https://lennerd42.github.io/posts/dark-souls-goals/#how-not-to-succeed-at-game-dev), which you should totally check out! But two consistent factors in this were perfectionism and scope creep. These two are a devious pair, as they amplify each other. With an ever-increasing scope and the desire that each game element be perfect, you create excellent conditions to be overwhelmed and to eventually fail.

I’ve heard the advice “start small” numerous times over the years. Each time, I’ve dismissed it as I thought I was above these small-scale projects. I, the Great Lennart, will not subject myself to these boring and trite creations. No, what I’m making will be high art, filled to the brim with content, great music, and gorgeous visuals. But inevitably, all of my large projects with their lofty ambitions eventually came crashing down. So, I swallowed my pride and started with a small idea that I’ve had for a while.

## Brittle Blocks

The idea was simple: a platformer where you place blocks to reach a goal. Before I started this project, I had been playing some *Ultimate Chicken Horse* with friends, which is a multiplayer platformer about, you guessed it, building various blocks to reach a goal. My game was going to be a simplified version of that, with the multiplayer elements removed.

I came up with the story of an explorer trying to reach an emerald in a cave. However, just before reaching it, they fall into black goop that has been chasing them. Inside the goop, they travel down instead of up and eventually reach a suppressed version of themselves. They offer love and are offered the emerald in return. What they’ve been chasing could be found somewhere else all along. I chose this story as it reflected my struggles and could be represented fairly easily in-game.

![Brittle Blocks Gameplay - Placing Blocks and Escaping Goop](/assets/img/posts/two-week-game/gameplay.png)
_Gameplay in 'Brittle Blocks': build blocks to escape the Goop_

With that, I just needed to work on the gameplay. I picked a standard control scheme of moving left/right and jumping. The player collects blocks from chests, which they have to use in a predetermined order. Left mouse builds the current blocks and right mouse rotates it before it’s placed. Blocks are placed at the mouse cursor, so the player has agency over where they are being placed. After a while, blocks start sinking, so the player has to advance quickly.

I chose to only create two levels for the game: the cave level and the goop level. The cave level has chests and various checkpoints. When a checkpoint is reached, the goop rises. This forces the player to keep building upwards towards the next chest and checkpoint. With each checkpoint, the blocks also start sinking faster and with a shorter delay to gradually increase difficulty.

## Each Session Counts

Not only did I set the scope intentionally small to begin with, but I also added another rule for myself to ensure it would stay that way: complete the game in 14 sessions. Basically, I gave myself 14 days to finish the project but didn’t force myself to work on it 14 days in a row. To remind myself of how many sessions I had left, I added a sticky note to the corkboard in my room - with 14 tally marks already present. Then, after I completed a session, I would erase one mark until there were none left. Had I added tally marks the usual way, I could’ve easily gone beyond the session limit. But erasing them made this impossible. After all, once all the marks were gone, there was nothing left to erase.

![Sticky Note labeled 'Brittle Blocks' with 14 tally marks, 5 have been erased](/assets/img/posts/two-week-game/tally_marks.jpg)
_A recreation of the original sticky note (the original has been erased, sadly)_

Another rule I set was to always share something after each session. I’m usually the type of person who keeps his projects a secret, out of fear of ridicule. This in itself is completely ridiculous, but it’s a difficult habit to break. After all, it’s far easier to say nothing than to show something. I pushed against this and always recorded a short snippet of what I added that session, be it a new feature, improved graphics, or sound effects. And guess what? No one ridiculed me. In fact, whenever there were responses, they were either positive or curious!

## Main Takeaways

So, what's the end result? That's simple to answer: **a small but satisfying success**!

I learned a lot from this quick development cycle. Scope creep wasn’t an issue. I actually managed to release a demo after session #8, so I could use the last few sessions to polish the game and incorporate player feedback. Releasing the game early and then building upon what I had instead of immediately abandoning it and rushing to the next project made me engage with the feedback I was receiving. Two members independently suggested a time trial mode, which I added during the last session as a send-off. I added a developer time that people could try to beat. It was actually quite fun, which surprised me.

## Where to go From Here?

My next project will be structured in a similar way. In fact, I’ve already been working on it for over 25 sessions as I’m writing this! This time, however, I have limited myself to 28 sessions so I can think a little bigger. It’s also going to be my first 3D game developed with Godot, so regardless of the outcome, it will teach me a lot. This time, I didn’t have such a clear vision going into the project, which forced me to think of a new way to prototype. Maybe I’ll talk about that in a future blog post. For now, I will retreat into the shadows and quietly work on my game. Wait, I should share my progress, shouldn’t I? Oopsie.

\- Lennart
