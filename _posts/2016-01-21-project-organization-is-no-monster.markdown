---
layout: post
title: Project Organization in Finding Monsters Adventures
description: "How the version control, branch model and development was done in Finding Monsters Adventures"
date: 2016-01-21 16:00:00
categories: articles
image:
  feature: "http://crocidb.com/assets/img/organization-no-monster.jpg"
---
**This article tells our experience versioning the code of Finding Monsters. All the opinions expressed here are mine and not necessarily reflect the studio opinions.*

I have been using Git with Unity projects for a few years now. When I joined Black River Studios, I helped the team implement Git on the current project at the time, Galaxy11: Invasion (First Black River Studios project). They were using the old Asset Server, but as the studio wanted to grow and improve its processes, Git seemed like a great choice, since most programmers on the team had already worked with it.

It was good for the project, but we had a few issues with merges and conflicts; sometimes we would miss Unity prefabs, inspector references or minor scene changes, things that happen if you have a lot of people working on binary files or huge text files.

But as Finding Monsters Adventure was about to start development, we decided to try Perforce due to recommendations of our leads and its own reputation across the gamedev industry. We put a lot of effort on changing our development flow by studying the new tool and setting up our new environment. Most of our developers had experience with Git and SVN, but barely with Perforce.

Back on G11: Invasion, we had a master branch, which we would never code directly into, and a lot of feature branches so that every person working on such feature would be checked out on the same branch. We would only merge when the feature was stable enough. Now on Perforce, we had to work on a single branch again, because we lacked enough knowledge on it to set a good workflow with streams (their branches). And I don’t think a centralized revision system should work well with branches anyway.

Of course there are other advantages of Perforce, apparently it works really well with binaries and giant files. And being centralized is not bad, really, just a very different approach than the one we were used to. For environment artists and game designers it was really cool to lock scene files or prefabs.

But that ended up not working quite well for us. Our adaptation was not so fine, so everybody working on exact the same code base wasn't good, designers were "messing" with unfinished features, programmers were changing tools that designers were working with, artists and tech artists were changing shaders while designers were renaming directories. Wow, such a complete mess. Everybody was uncomfortable with that. Then we decided that the risks to switch back to Git in the middle of the project were worthier than keep bashing those streams and workspaces in the head. Giving up on a tool your team knows well to another different one which nobody in the team has experience, only because other people say it’s better, may be [considered harmful](https://en.wikipedia.org/wiki/Considered_harmful).

We knew that we had to improve our workflow from the last project, even more because our team was growing. All people in the team would have access to the project, but it shouldn't become a total mess with long-lived parallel branches (those are the worst to merge), or change-scene-party (where everybody feel the desire to change the same scene on a parallel branch).

### Branch Model

We improved the model from our last project and added a new layer.

![Comparison between G11: Invation and Finding Monsters branch model](http://i.imgur.com/1LepFwY.png)

On Finding Monsters Adventures, our master branch had only stable versions. That means you could check out that branch and play the game and it was sure to run well. Then we had one branch for dev, one for art and one for game designers. So programmers were only allowed to check stuff in the dev branch, artists in the art one and finally game designers in the gd branch. But that doesn’t mean that everybody within a given area should be working on the same branch.

Note that the feature branches are still there. One could create a new branch whenever needed, do whatever within it, then merge in its respective area branch. Or maybe work directly in the area branch. That way we knew that no game designer would be upset if we change all the AI Tool in the middle of their editing. No more pull-surprises.

Every one or two days one person would be responsible to merging area branches—usually our Lead Engineer, or myself—to master and then back again to them, so all the four branches would be the same. This person needed to know a little bit of whatever was happening to the project in all areas. Sounds a little too time-consuming, but that really wasn’t.

### Communication

When we can’t lock resources, or automatically tell others what we’re working on, we need to let other people know that in some other way. You may think at first that it should already work, no matter whether your tool supports locking or not. Sure, but things don’t always work that way.

We had to put some effort on improving our team communication to make this work better. From ask-anyone-whos-working-on-something to the ~innovative~ Tea Box Locking™ technique, where there was one tea box for every level in Finding Monsters and every person that would edit the level should bring the token box to their desk. Important to note that maybe two people could edit the same level at the same time, but they would need to talk first.

![The Tea Box Locking™ token technique](http://i.imgur.com/M2OXJRn.jpg)

*The Tea Box Locking™ token technique. If I got those token boxes on my desk, I should probably be editing levels 1-4 and 1-7, so don’t touch them without talking to me first. Thanks.*

Asking the right people before start working on a resource shouldn’t be too complicated, come on. That even helps the team to know better about the project, so everybody knows exactly where the project is at.

We even created a tool to improve the token grabbing, the Asset Manager System, but we ended up sticking with the Tea Box, because it was created at the end of the project development and we didn’t have the time to get used to that, but it was very useful and we’re likely to use that tool on the next project.

![Our super-easy Asset Manager System](http://i.imgur.com/zSYvrOC.png)
*Our super-easy Asset Manager System, just drag the scenes, or whatever you’re going to work on, if someone else is using it, you can click “Watch Asset” and get notified when it’s free.*

### Conclusion

We were always trying to improve, so even though Git was satisfying, we decided to try an industry standard solution to solve our problem. It seemed better, but happens that it wasn’t working well for us, in fact it ended up being worse. After struggling with the tool for a while, we decided to switch in the middle of the development. It’s usually not recommended, there are risks to take doing such change, dozens of people need to stop working for a few hours and then start again in a completely different environment. We had to give a couple of PowerUp! presentations (our internal weekly talks about whatever technical subjects one wants to speak and teach others) to bring people to our new development process.

In the end, our risks were worth it. This new workflow using git improved our productivity. There was always a stable branch to start something new from; everybody was confident enough that there would be no major problems merging stuff from here to there, hence more freedom; and finally, because everybody now was working on a project they knew more about or, as people here like to say, “everybody is now working on the same project”.

What about Perforce? Well, the art team still uses it to deal with the art sources, and it’s pretty good. Processes and tools work differently for different teams, use what suits better to your team.

