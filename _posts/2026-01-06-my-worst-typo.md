---
layout: post
title: How a Typo Broke my Linux Installation
description: The right slash in the wrong place can make all the difference.
tags: [linux]
categories: [Stories]
date: 2026-01-06 17:24 +0100
---

## How It All Happened

"You are now in emergency mode". That was about the last thing I wanted to see from my Linux Mint installation. But here it was, small white text on a black background. Afterward, all I could do was to enter my password in an endless loop. After entering it, Mint would wait about 15 seconds before prompting me to enter it again. And there was nothing else I could do. Distraught, my dad and I gave it our best shot to fix the issue, but it was just too late in the evening to deal with problems like that. So I went to sleep, unhappy that my two weeks of setup resulted in this, of all things.

Furthermore, I had absolutely no idea what had just gone wrong. One minute, I was happily writing a different post for this blog; the next minute, everything was broken. I had lots of theories: GRUB was acting up, the Nvidia drivers were causing issues, OpenRazer might be to blame. Maybe Docker messed something up. But all of those were just theories. If I had no idea what caused this, how was I to fix it?

The next day, we gave it another shot. We booted from the USB stick I had used to install Linux Mint and were finally able to use a command line. We tried some things Google Gemini had suggested - maybe it would give us just what we needed. But no dice. Linux Mint insisted that something was wrong. Strangely, it complained about not finding `bin/bash`, among other essential files. How could they have gone missing?

## Fixing Linux Mint

That's when I noticed something. The root folder (`/`) of my live USB session had a `bin` folder, but my actual Mint installation, which I had mounted to `mnt`, had not. Why was it missing? And what even was the `bin` folder exactly? Looking into the USB session's `bin` folder, it apparently contained all the installed programs, among other things I didn't quite recognize. As it turns out, it's merely a link to another folder `/usr/bin`. This folder still existed in my Mint installation, so maybe all I had to do was add the `bin` folder back?

I did this via Mint's graphical file browser. I love the command line as much as the next guy, but sometimes, I find it easier to use a GUI for certain tasks. After ensuring that the newly created `bin` folder was pointing to `/usr/bin`, I rebooted the computer and held my breath as I selected Mint from GRUB's menu. And lo and behold, it worked. It booted like nothing had happened, and, most importantly, all my programs and files were still there. Problem solved! But how had this happened in the first place?

## The Root Cause

At first, I chalked it up to some program messing things up without my knowledge. But as I was taking a well-deserved break, I thought of something. Before everything had gone to hell, I had used `mv` to learn how to move files and folders from the command line. And I distinctly remember moving a folder called `bin`, which resided in my downloads of the Android command-line tools. Surely, I didn't just move the wrong `bin` folder. I wouldn't be stupid enough to do something like that, right?

I held the up key in my terminal for a while. Luckily, my command history was still intact. And before long, I saw this:

`sudo mv /bin /latest`

There it was. Plain as day. The command to rename the `bin` folder residing in `/` to `latest`. What's funny is that right before that, I had attempted to do the same rename, but without admin privileges:

`mv /bin /latest`

Naturally, the console gave me an error, complaining about missing permissions. After all, not just anyone should move files in the root directory:

`mv: cannot move '/bin' to '/latest': Permission denied`

But being the fool I was, I simply ignored this and re-ran the command as root again, dooming my Linux installation to a painful death the next time it booted up. What I really wanted to do was run this command:

`mv bin latest`

This would move the `bin` folder from the folder I was in into the `latest` folder. And it wouldn't even require root privileges.

## Takeaways and Closing Words

In the end, all I can really do is laugh at my foolishness. While the resulting problems had caused me a lot of stress, I also learned how to troubleshoot my installation, should things go south again. What's even better: I was writing an article for this very blog before it all happened. It was going to be my first (proper) article, a guide on setting up and using an Android emulator using only the command line. But now, my reckless use of the command line has given me a much better topic for my first blog post!

Thank you very much for reading all of this. I plan to publish more articles on here about all sorts of things. Not sure about what exactly, but mostly software & game development-related things. If you liked what you just read, leave a comment! All you need is a GitHub account. Alternatively, you can shoot me a message in my [Discord server](https://www.discord.gg/Q27rN7b).
