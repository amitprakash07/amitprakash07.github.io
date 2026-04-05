---
layout: post
title: "Game engine coursework: Visual Studio solution setup and asset build system"
date: 2014-10-05
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/assetbuildsystem"
---
## Visual Studio Solution Setup and Asset Build System

I, many times, encoutered with various online samples or codebase of some of the famous game Engine especially Unreal Engine. The build system of Unreal System seems complex but same time decoupled - functionality are separated out in different projects/static library. With this assignment, I understood many complex features of the visual studio which are very helpful in bulding such solutions.

In the earlier semesters, we wrote and developed libraries for a Game Engine subsystems like Physics system, Collision System and others but we didn't worked deeply on the organization and maintainance of the code. In this assignment, I learnt greatly about the organization of the code; how to generate and maintain in a way that their won't be any extraneous files in the shipped game.

The solution was designed in a way that all the Code, Assets are separated and be placed in different folder. Moreover, the solution also separated out the temporary files - files which can be generated anytime using the Code and Assets.

The other good part of the project is to learn how to use and create the property sheets. Property sheets are XML files contanes cutomized settings for the visual studio. Visual Studio allows to add different property sheets to the different configuration for each target which makes the solution more customizable. Each project can have as many project sheet but the order of adding the sheet is important if the property sheets are dependent to each other.

Another interesting and important element of the project was Asset Build System and its build. Currently, Asset Build System just copies the Assets (Shader Files) from the Asset folder to the temp folder corresponding to the configuration and target system. Asset Build System uses environment variable (not system environment variable) meant for visual studio to look for the Asset source folder and Build Asset target folder. Other unique part of the Asset System was its build using custome build feature of the Visual Studio. We have created a separate project (BuildAssets) whose custom build is only dependent on the execution of Asset Build System with argument of the assets. The good part of this feature as it is independent of the game and can be run on different thread to build the assets in the game.

Other learning part of this project was the design and organization of the projects in the visual studio using Solution folder to decouple the functionality of the sub-systems.

The main output of the project was very simple - just to draw a triangle on screen with a provided vertices and sample shaders. But the tricky part was to genrate using both DirectX and OpenGL graphics API using lucid interface hiding the complex behavior of both API. Output of the program was something like below image.

Another interesting aspect of the project was to generate the required file to run the game in separate folder by copying all required files from the configuration + target folder to a game folder with custom build option of the game project.

With almost 10 hours of work, I was able to finish the project. [Please click here to download the executable file](https://www.dropbox.com/s/8nplzh8syexu5j5/AssetBuildSystem.zip?dl=1).