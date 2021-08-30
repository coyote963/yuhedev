---
title: "Discord Bot"
date: 2019-12-20T10:00:18-05:00
draft: false
toc: false
images:
tags: 
  - boring-man
  - discord.py
  - programming
---

Documenting the process of creating a discord bot.

## So I wanted a discord bot... ##

All these times, I have seen people talking about discord bots and using them to do all sorts of amazing things, I have felt left in the dark because the last time I tried using discord.py's library I failed to wrap my head around everything.

That was a time when my ambitions didn't measure against my ability to read documentation. This time was different, and I am proud to announce my new discord bot: Coybot

## How I made it ##

Well first I was looking online for some sample projects so I could see what kind of code was boilerplate, and I could just copy. This led me to a couple libraries like [this](https://github.com/SourSpoon/Discord.py-Template). I noticed that this boilerplate was really extensive and seemed to be a overshoot of the small problem that I had. So then I noticed in the readme of that repo, that there was a way to generate a template for this exact purpose!

Great, so I did `python -m discord newbot coybot` and the bot was made! I wanted everything to be in best practices so I immediately made a cog, then ran into the issue of not being able to load the cog in with `load_extension(cog)`, thats when my shaky understanding of python's module system came back to bite me. After a few minutes of googling, I realized that

0. You don't just type the name of the cog
1. You don't load the cog object into the config.py
2. You don't type a relative path in there either

Welp finally the cog had loaded and all was good!

Now I had code that used webhooks to send messages to my discord server, so it was really quite easy to convert that code into callable functions that simply required database clients, but that raised the final issue which was: Where was the best place to instantiate my database connection?

I finally put it in the `init` method, put the database uri in a secrets folder and called it a day. With some debugging everything worked fine and dandy.

## Conclusions ##

The groundwork for the bot has been set, I just have to hammer out a bunch of commands. With a system like cogs in place, it nicely packages everything in your filestructure. I am beginning to think how best to separate my commands, and whether that is even necessary.

More updates to come