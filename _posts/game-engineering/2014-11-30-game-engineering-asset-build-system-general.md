---
layout: post
title: "Game engineering: Asset build system — general approach"
date: 2014-11-30
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/robust-asset-build-system"
---
## Asset Build System - More General way for Building Assets

This assignment is intended to extend the basic build asset build system to a more asset oriented build system. Earlier, our asset build system was using a command line arguments for the asset list; using which it builds (copy) it from one Authored Asset directory to the build Asset dirtectory. In this project, we created other asset -type -centric build system - Mesh Builder for building meshes, common assets using generic builder and shaders using shader builder. Although the build systems were different, the entry point for building the assets were same which was accomplished using interface.

In this design, we created an assetToBuild.lua file, as per our conveninet format, which contains basically three information -

* Asset Relative source path

* Asset Relative Target Path

* Asset Builder relative path

My format for this file is like below image.

The reason, I chose this format as it was simple to read and it contains concise but sufficient information for building the asset as per the asset type - builder to use for building the asset and the source and target file name information about the asset. I kept this file under the Scripts solution folder and in AssetBuilder project in the solution explorer. As its a script, script solution folder is best candidate for including in it. Moreover, it is being used basically by AssetBuilder, so I just added in the AssetBuilder project too.

The working of Asset Builder is very interesting - it uses a builder helper interface project which has template object which calls its own build logic. Any builder have to extend this builder helper and implement its own build function which will be finally called by BuildAsset function from Lua file. Below image may give a brief picture of the flow of the build system.

I also changed the dependency for the AssetBuild System according to its dependency rather than adding all. Below is my dependency for AssetBuild System. Build Assets Project mainly calls AssetBuilder, so I just added the dependency to it. For AssetBuilder, which only going to call Generic Builder and Mesh Builder (which will underneath call BuilderHelper) - that's the reason for adding dependency it to GenericBuilder and MeshBuilder.

Moreover, As BuildAsset only needs to call AssetBuilder - AsetBuilder is taking care of loading the AssetsToBuild.lua from the script directory. Below is my custom build setup for the BuildAssets project

With right tools, it becomes very easy to finish up the tasks. In the previous, assignment I used a separate test project to check the correct loading of the Lua file which was okay but not a good approach - needs building and then refactoring in the new project. In this assignment , I used SciTE which is very useful for Lua debugging. I wrote and tested my Lua function in here which hardly took 10 mins of mine to figure out the correct syntax and inbuilt functions for lua.

Although, I used SciTe for writing the Lua code, but I found sometimes, at least in my case, inbuild function doesn't work always. I used a funtion table.getn(tableName), gives the length of the table, was working perfectly in SciTE but was not working while accessing it from inside the project.

This assignment would have been less tedious, if I had structured my Mesh class in a way so that I can add multiple meshes. This assignment, I made a separate class, Scene, which basically contains all the meshes and a drawScene function which underneath calls drawMesh() function for each mess in the scene. Now, my render functions just calls Scene::drawScene()

As per the submission checklist, I debugged the meshBuilder also to check it's working correctly or not. Below is couple of screenshots of it.

This assignmentwas fairly easy and less tedious -but due to restructuring the code for Mesh structure, it took little more time of me. With almost 4 hours of work and an hour for write-up, I finally got the same result for the openGL and Direct3D like below.

Here is my [DirectX](https://www.dropbox.com/s/go34i4vnz2h15bn/Assignment05_DirectX.zip?dl=1) and [OpenGL](https://www.dropbox.com/s/czw0o4zhvvvdfs5/Assignment05_OpenGL.zip?dl=1) link to the executables.