---
title: "Postmortem: Phosphorus Dating"
description: "A quick postmortem of Phosphorus Dating, an awarded dating simulation game created for JS13K competition."
date: 2016-10-17T08:58:51+02:00
---

Last August started the 5th edition of the [JS13K game competition](http://js13kgames.com), where you have to create a complete game in 30 days, using javascript with 13k size budget for the whole project, including assets. I joined the competition and managed to ship a game called **Phosphorus Dating**, some kind of dating manager/simulation game. The game got the *8th place* in the **Desktop** category of the competition, and the *20th place* in the **Community Awards** category.

> Phosphorus Dating is a game about Technology and Relationships. Set in
> 1996, a lot of people is amazed by Internet and want to try to find
> someone to date online. Phosphorus Dating website has a very smart
> algorithm to find a match for anybody based on their gender, sexual
> orientation, age and interests, but for some reason this algorithm is
> not working as expected. 
> 
> You are **Cecilia Dent** and have to find the right matches for these
> people while the algorithm is being fixed.

You can [play it here](http://gamejolt.com/games/phosphorus-dating/189626).
	
[![Phosphorus Dating](http://i.imgur.com/hHkUlue.png)](http://gamejolt.com/games/phosphorus-dating/189626)

When I decided to join the competition, about one week before it started, I was thinking of making some sort of procedural-dungeon-rogue-like. I even started some small prototype, but I was surprised with the theme: **Glitch**. I spent two weeks trying to think of a way to merge the theme with my former game idea. When I finally decided to drop the rogue-like, I had a simpler idea of a manager-like game, with some narrative. All the game came up in just two nights while trying to sleep, thanks Google Keep for helping me not forgetting anything.

Probably inspired by the great *Her Story*, this game happens in a old computer terminal, with the Windows 95 look and feel. You have to match couples based on people’s gender, sexual orientation, age and interests. That’s the basic game mechanics.

# What went right

## Simple game idea

Breaking the game idea into elements, that would be something like:

- Character generation
- Match evaluation
- Game progress gameplay (conditions for winning and losing)

Generation was the simplest thing in the game. I just had to feed it with a names, last names and a bunch of bio phrases about certain interests and it randomly combines them forming a person. Is it convincing enough? Well, it is, taking into account the amount of work I had on it.

Evaluating subjective things like a couple dating is something more complicated because humans are very unpredictable. So I used a lot of randomization into that too, so it’s never going to happen the same with equal inputs, but at the same time, it’s pretty easy to feel what will happen. Gender and Sex Orientation are the only certain aspects that, if you chosen wrongly, match won’t be good. The bio texts has to be as clear as possible, but still sounding as if a human had written that, example: something as “we should get out for a beer” in a profile will be against “no alcohol for me” in the other.

The age is sometimes a penalizer, if you try to match a 18 year-old person with a 45 one. But still there’s a random to balance that.

I think that it feels organic and not too mechanical, I liked this result.

> <i>Really innovative!</i>
> 
> -- Pedro Fortuna

## Subtle story

There is a very subtle story and you don’t need to understand it to enjoy the game. I could have done it without any story or narrative, but I wanted this game to be a little deeper than an arcade. A better experience. Short but complete.

I won’t spoil the underlying narrative, but it only happens in the beginning of the game, before “loading the graphic interface”, and at the end, when you either fail or complete the unspoken mission. There are clues during the gameplay that will help you understand better what’s actually happening.

I liked this and all the tests I did with people playing it were interesting. However I wish I had more people and time to test it.

> <i>I really like these types of games. It was fun trying to match people
> up and reading about their interests and what not. Its a cool game
> with a good story too!</i>
> 
> -- Jupiter Hadley

## Art direction

The good part of the retro computer interface is that is pretty simple to implement using HTML and CSS. Even my five-years-dated CSS did the job. And the bonus was that I didn’t need to use images, which helps a lot in a competition where you have only 13kb to make a game.

[![Phosphorus Dating screenshot](http://i.imgur.com/UcOO6kD.png)>](http://gamejolt.com/games/phosphorus-dating/189626)

> <i>I love the startup console text stuff - made me feel warmly nostalgic
> about the BBS era, with ansi graphics goodness. In truth I wish it
> stayed in text mode, but the windows 3.1 style gui was also really
> amazingly done too. Nice work.</i>
> 
> -- Christer Kaitila

# What went wrong

## No sounds

I didn’t even bother with sounds because I knew that I couldn’t afford space for assets. I thought that using all the remaining bytes to feed more texts for the game was a better idea. I don’t regret my decision, but a few sound effects for feedbacks would make the game feel way better.

## All the code in one file

When I started to code I was in a gamejam mood, just as if I had only a couple of days to make the project, whereas I actually had two weeks left. So I didn’t use any node-ish tool. Just created one HTML file, one CSS and finally one JS. Opened Visual Studio Code (which now is possibly the best lightweight code editor) and started hacking.

One week later, working with only one file was getting annoying for me. Now I know I should have taken at least one day to setup an environment with tools to merge all the files and minify them automatically. It’s complicated to work with 1000-lines-source file, even worse when it’s a dynamic language and intellisense can’t help you that much.

The code is open in my GitHub and you can [check it here](https://github.com/CrociDB/phosphorus-dating).

## Problems remembering characters

Since the beginning I felt one thing wasn’t good: right after reading the characters info, I wouldn’t remember them. So I tried to do some things to make them a little more remarkable. Creating procedural avatars would be the first choice, that would make us associate their attributes with their “faces” and names. Unfortunately that sounded too complex and long for me to do with only few days and very little size budget. A friend of mine even offered himself to do that, but I had to drop that.

Finally I changed a little bit how the week system worked. At first, every week new characters were generated, so I realized that if the same characters were there until you take them out, either by matching with someone else or dismissing them (which brings new people in), that worked better for us to associate people with their names.

# Conclusion

It was a great experience for me to make this game. There’s been some time since the last gamejam or competition I participated. Even though I work creating games everyday, this feels so good to create a complete game experience in just a couple of weeks. I learned a lot about javascript and game design.

Do you have any feedback? I'd love to hear what you thought about the game. I'm even considering making a "complete" version of it if I have enough interesting feedbacks. Thanks!


[![Phosphorus Dating logo](http://i.imgur.com/VPX2YeU.png)](http://gamejolt.com/games/phosphorus-dating/189626)