---
layout: post
title: "Project coursework: Analysis System"
date: 2015-11-07
tags:
  - unity
  - analytics
  - game-development
source_wix: "https://amitprakash07.wixsite.com/home/post/2015/11/07/analysis-system"
---

This is an analysis system which I created to analyse the tenure of the played game by the various users.  It's on its own is a small analytics system. This system only creates the game process and captures the start time of the game. As it holds the process for the game, it can easily capture the end time of the game.

One can esily argue, why to make a separate process for this behavior as we can do same thing with Unity API. But the problem is that Unity API will only works when game is closed normally i.e. by using exit options of the game. If somebody uses system Alt-F4 option, it won't be able to capture the exit time. But with my system, evenif user uses Alt-F4, it will kill only game process but not the anlytics process which will exit once it captures the termination time of the game process and sending an email to the game account along with its game duration.

![]({{ '/assets/img/blog/2015/analysis-system/analysis-system.png' | relative_url }})
