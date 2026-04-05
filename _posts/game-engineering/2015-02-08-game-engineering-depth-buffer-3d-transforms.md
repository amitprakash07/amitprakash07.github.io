---
layout: post
title: "Game engine coursework: Depth buffer and 3D transforms"
date: 2015-02-08
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/3d"
---
Adding Depth -Buffer

This assignment was interesting as the output changed to the 3D. In this assignment, we intend to add the depth (z- value) to the positions of the meshes. Earlier we were passing the position for the mesh in the screeen coordiante system. We didn't had to do any transformation to and from different coordinate system. Opposed to the previous float uniforms, now we are using three matrix uniforms as below:

1. Local-To-World Transform Matrix: Authored Asset are authored in its own coordinate system (local coordinate system). Origin of the asset may vary to the requirement of the asset in the game and might not be at the center of the mesh. If we need multiple instance of same asset/mesh at different places; we don't have to create another one. We can use the same asset in the game by just changing its position in the virtual world. For this transformation i.e. from object local coordinate system to the world system, we use Local To world Matrix.

2. World-to-View : The object/mesh, we create in the game and change its position with different mechanic is carried in the world Coordinate system. However, we see things through a camera usually a pin-hole camera. When the camera moves in a particular direction, the whole world moves in oppsite direction relative to the camera. As we look from the camera, we need to change the position of everything (in the world) to the camera view so that the objects position is relative to the camera. For this, we use World-To View Transformation Matrix.

3. View-to-Screen: As we see, everything on a screen - a 2D screen, we need some sort of transformation from Camera to Screen coordinates. To achieve this, we project all the object in view frustrum defined by the camera on the screen. Moreover, screen coordinates is in the range of [0,1]. View-To-Screen matrix transforms the view coordinate objects to the screen coordinates.

There is couple of ways to draw object in 3D on the screen in computer graphics. One of them is painter's algorithm - in this algorithm we draw objects in a particular order. Objects in the front are drawn last and vice-versa. This algorithm is not effective as in any case we are drawing the complete object - although it can't be seen due tot he objects in front. To improve performance, now a days usuallly, we use depth buffer test to determine what to draw and what to not. There are various options available in DirectX/OpenGL API to perform the test. In our implementation we used "LessThanOrEqualTo" function for comparing the depth buffer. Objects whose positions have less or equal depth to the entries in depth buffer will be drawn on the screen and rest will be ignored.

As the screen coordiantes are always in the range of [0,1] - we are clearing the all the object from the screen coordinate system by clearing all the objects with the depth of 1. It means that everything with depth of 1 and less than 1 will be cleared.

To chack the z-buffer, we needed to change the human readable file to add the depth value - z coordinate. My human readable file and its binary file looks like below images. The z-values in the human readable files are in local coordinate sytem/object ccordiante system. It gaves a depth of the vertices from the origin of the object/mesh itself.

Interesting part of this assignment was the part where we have to design and implement a camera. I opted to create a class for creating the camera and kept it in Engine Project - as all of my objects and controller are in Engine. Moreover, Camera is also an movable object so I chose to make it in the Engine. My Camera class structure looks like below:

As I considered Camera as an Object which required movement too, thus I created CameraController and attached it in the game.cpp like below. When I am calling my drawScene() - which contains list of cameras (as well as game Objects) in the scene uses an active camera to set the uniforms in the effect.

Below is my what drawScene() function looks like:

And I create my Camera ("MainCamera") in the game.cpp and attached the camera controller to it. I also made an option to create as many camera we want in the scene but scene will use only one camera which is made active during the draw call.

This assignment was fairly easy compared to all of the previous assignment as most of the things was already done, we just need to add couple of parameters. Moreover, I had already wrote the reading z values from the human readable file in the previous assignments - I just uncommented them this time. I haven't faced any problem in this assignment except color issue which I got clarified in the class and fixed a small piece of code while writing to the binary file.

Implementation wise, this assignment took almost 5 hours, although write-up took almost 1 and 1/2 hour. As expected, I got the desired results as below:

Here is the [DirectX](https://www.dropbox.com/s/0fc1zuw6e80owh3/Assignment10_DirectX_Release.zip?dl=1) and [OpenGL](https://www.dropbox.com/s/tgudfb80d87aeps/Assignment10_OpenGL_Release.zip?dl=1) link to the executable. Controls 'WASD' for Camera Movement and Arrow Keys for Cube Movement.