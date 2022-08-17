---
title: "A story about a mobile game's economy and git blame"
description: "Someone broke a mobile social game economy and I had to find out who did it and why."
date: 2022-08-16T08:58:51+02:00
tags:
 - thoughts
 - story
---

Years ago, back when people still used personal computers, Social Games were a thing. You would log into Facebook on your desktop to play video games. Mostly farm management or bejeweled-like puzzle games. You'd have to log in multiple times a day because you didn't want to miss the perfect moment to harvest your pumpkins or let the cool presents that your friends sent you rotten.

You open the game, and the browser stutters a bit, after all, the Flash Plugin is a bit heavy for this computer, huh? There are two types of currency in it, the *soft currency* and the *hard currency*. The former you can get by completing quests in-game, selling stuff to an NPC, and opening gifts from your friends. It allows you to buy most of the stuff you need to keep playing day after day. The latter, however, you pretty much only get by putting in real money, or maybe from some very specific tasks that *the developers want you to do*. And it's really valuable because it makes you stand out from other players, it will make you become a god.

Back then I worked on a social game like that and the currencies were taken very seriously. So many players depending on it to come back and play every day, many times a day. A friend of mine, who worked more directly in the mobile version of the game–a version that, instead of opening Facebook to play in it, you would log into Facebook from within the mobile game app and have pretty much the same experience natively on your phone–, called me asking for a very specific thing:

> "Hey, Bruno. Can you help me figure out _git blame_? Someone broke the game economy and I'm being blamed, I want to make sure it wasn't me."

He knew the specific line of code that "broke the economy", so I went there and just ran the command with him:

`git blame -L 201,205 game_file.cs`
 
My surprise was to see *my name* on it. I broke it. He looked at me and:

> "Did you... did you commit it?"

Yes. I was a bit embarrassed to admit it, but it was a problem with a feature I implemented not long before that. A problem that I didn't catch and nobody else in test caught either.

The so-called "economy breaking" problem was that people were getting so much hard currency every day and then nobody was purchasing it anymore. 

I remember the task I implemented was something like _"create a quest that gives the player X hard currency when they connect their Facebook account"_. I never worked on that game before that point (and as a matter of fact, didn't work on it after either). I skimmed through the code for the Facebook integration and there was a callback called something like `OnFacebookAccountConnect`. That sounded promising, and that's where I plugged in the trigger for the quest that I created. I tested and it worked perfectly. Committed and sent for testing. It was approved and in no time the patch was in production.

Turns out the callback wasn't for the account connection, but _authentication_. So every time you opened the app, it would auth your Facebook account and give you more hard currency.

The game had limited play sessions per hour, which was (and still is) a cheap way of limiting the amount of time you play at once and forcing you to come later to play more. The hard currency could buy you more chances to play. This puts the player in the decision of either playing many short sessions a day or paying for longer play sessions.

I think I made a few people happy, even though the company made fewer bucks during that patch. It was fixed in no time, though. No major problems were caused.
