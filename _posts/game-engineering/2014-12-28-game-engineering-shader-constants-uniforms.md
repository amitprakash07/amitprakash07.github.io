---
layout: post
title: "Game engine coursework: Shader constants (uniforms) and moving objects"
date: 2014-12-28
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/shaderconstants"
redirect_to_wix: true
---
## Shader Constants - Passing additional Data to the Shaders

The purpose of this assignment is to understand the usability/flexibility of the shader constants to move objects (GameObjects) in the game. These constants are termed as uniform in terms of shader language as they are constant for every draw call. The goal of the assignment was to use this uniform to move objects in the game using keyboard controls.

There are also other ways to move the objects in the game. First one, which I think of was as I didn't have good knowledge of shaders, is to update the vertex buffer data i.e. once the game gets the input, the gameobject should update the mesh vertex data using span of time for the frame. And change the buffer after it. As per our design, buffers (index and vertex) are loaded at the loading of mesh itself. With this design there is expensive computation and extensive resource utilization involved in this process. The reason for this computation and resources is that every time whenever we want to update the buffer, we need to unlock and lock the buffers each time which is not a good practice in this case.

Use of uniform is simpler and faster as we just pass an extra constant "position offset"; in any case we are passing the position to the vertex shader, thus it won't be high on computation to pass an extra constant. We achieve this using constant table features of the shading languages.

The other interesting aspect of the project which was not a requirement was to think and implement a high level design for the engine so that it efficiently keeps track of everything and update accordingly. Except Math class, I brought few of the classes, GameObject, MessagingSystem, GameObjectControllers and some utility classes, from the previous semester. I designed in a way so that I don't have to change graphics classes for rendering objects. My gameobject looks like

Moreover, I used messaging system for input call back events. I used the windows callback to take the input and pass relevant data to the registered object controller of the game object - so that game object controller can implement custom logic for different gameobject.

This project was fairly easy if I haven't brought up the things from my previous semester. It took me while to integrate everything altogether as I intended to use SharedPointer. But the only problem I faced using the Messaging System as it needs the Shared Pointer of a base pointer opposed to a void*. I used a very simple RTTI to achieve this. But even using this I was unable to cast the SharedPointer from one to other type , although both were inheriting the RTTI interface.

On the other hand, as I already designed my code in previous assignment to handle the list of all the meshes and effects, I didn't have to do any changes in graphics code.

Moreover, as John-Paul gave an idea about how to use shared effects and meshes to the gameobjects, I did it in the same way. As I was maintaing a list for them, I just created an map with the effect name and the shader files, mesh name and mesh file, little bit similar to Unity engine.

Moreover, I tried to mimick Unity, and created a scene class which is not a static class so that game can have multiple scenes and can switch between wach other as I am maintaining a list of scenes to with a map <name, scene>. As of now, it can switch between scenes by just setting a bool to the scene - moreover, one can set only one scene to be renderable, like any engine.

Another thing, I tried to micmick from Unity or Unreal is to add multiple controller to perform same thing. I used arrow as well as WASD keys to move the game Object

I gave significant amount of time in this assignment. Integrating things from previous semester almost took 9-10 hours. Designing and implementing RTTI took like an hour or two. Only thing which took huge amount of time was to cast the shared pointer objects to SharedPointer of RTTI - which almost took 6-7 hours but still I was unable to figure it out until John-Paul suggested a way. I still need to make it work which I intended to do in between next assignment.

I faced couple of issue, although interesting, in this assignment. First bug, I encounterd when at runtime I was getting an error that "d3dx9shader.h" not found. I checked the whole solution and with trial and error fixed the problem. But with John-Paul's explanation, I understood the reson behind it. The reason was if we are trying to add header file in a project which contains an inclusion of header file (like d3dx9shader.h) in a different project, we need to add the path to that project too (standard files like <iostream>, <string> works without this as their path automatically gets added when the project gets created in VC++ directories list).

Another roadblock occured to me, when I refactored a bit in the last moment to avoid memory leaks, was that the game was not rendering anything in Direct3D mode. I dubugged very closely step by step and found that theDirect3D primitive draw call was not returning S_OK. I thought something is wrong with the memory in a way that they are getting dereferenced when functions are calling between projects. I debugged a lot for finding this bug. Later on I cross checked the memory address to make sure everything is initialized or not. I cross referenced with my previous working commit and realized I did a very simplistic but a blunder mistake while refactoring. I was binding the device buffers first and then the shaders. It was really hard to debug this and it took almost 5-6 hours of mine but it was worth as it gave me little more working idea of DirectX (lesson learned - don't come to any conclusion until verified everything and don't try something in the end of finish line).

Finally, I got the output as expected like below.

Here is the [DirectX](https://www.dropbox.com/s/uyowu44rsxsuf5r/Assignment07DirectXRelease.zip?dl=1) and [OpenGl](https://www.dropbox.com/s/39jpbhsldyfh5up/Assignment07OpenGL_Release.zip?dl=1) link to the executable. Controls 'WASD' and Arrow Keys