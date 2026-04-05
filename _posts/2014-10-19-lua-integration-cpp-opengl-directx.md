---
layout: post
title: "Lua integration with C++, OpenGL, and DirectX"
date: 2014-10-19
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/lua-integration-with-c"
---
## Lua Integration with C++, OpenGL and DirectX Difference

In this project, I integrated Lua with C++. Lua is one of the most used scripting language used in Game Industry. Lua, usually, used for providing the custom informaion to the game rather to the game engine. This gives flexibility to change certain settings or information to the game such as custom assets for different platforms. The good part of the language 'Lua' is its integration - Lua can be completely integrated in a sense that Lua program can call C functions and vice versa - which makes it best for the development pipeline.

In this project, I used Lua for loading the Assets (mainly shaders) for cross platform development. Lua file was used in a way that it will call c++ functions to load (mainly copying from Asset folder to the game folder). To achieve this, main function of the Lua file was called from the C++ and rest of the part was taken care by Lua-C++ binding functions.

Good Part of the project was that we build the complete Lua interpreter using the Lua source file as a static library and used appropriately in depending projects.

We have used a lua script file, BuildScript.lua which contains mainly two things - a lua function (BuildAsset) and various C++ functions to do some windows specific tasks. One can call C/C++ functions from Lua by binding/registering C/C++ functions using Lua APIs. I caught up in one issue while doing it as Lua registering function only takes static functions (and not member functions). The reason is Lua register function is unaware of the instances of the user defined class variable, even if we are trying to pass using 'this' instance.

I will try to brief the use of Lua function using below Image. Our project was only BuildAssets which calls the AssetBuilder which is written in such a way that it will call the lua function - among which some of the Lua Functions are registered to C/C++ functions which get called consequentially.

Another aspect of the project was to understand the dependency and generalizing the same whild developing for cross platforms. As we all know, OpenGL and DirectX have subtle differences like (and most important) coordinate system orientaion, screen coordinate system and others. While OpenGL uses right coordinate system, DirectX uses left coordinate system. It becomes very important to handle such dependencies while developing for cross platforms. It's good to use an interface, as we did, and change the underlying code for the corresponding platform.

After almost 6-7 hours of work, I was able to integrate Lua and change platform specific code to generate a square as like below.

Here is the [Direct3D](https://www.dropbox.com/s/ltdc7dyn3aws5zw/LuaIntegrationDirectX.zip?dl=1) and [OpenGL](https://www.dropbox.com/s/mbrkn9m455elvqk/LuaIntegration_OpenGL.zip?dl=1) release executables.