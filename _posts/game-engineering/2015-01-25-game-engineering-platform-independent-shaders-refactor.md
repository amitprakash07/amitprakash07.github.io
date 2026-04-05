---
layout: post
title: "Game engineering: Platform-independent shaders refactor"
date: 2015-01-25
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/pishaders"
---
## More Platform Independent Refactoring

There was two part of this assignment. First to write platform independent render() function in Graphics and other was too write platform independent shader (to a certain extent) code. First part was really easy for me as my render() function was only calling one function to draw everything. All I had to do was to make the stub/platform functions so that I can hide the platform specific implementation from the render() function. Now my render function looks like below - no platform specific code.

Render() function contains functions which have platform dependent implementation. Sequence of functions I used in the Render() function are as below:

clear() - This function contains the platform (OpenGl/DirectX) specific code to clear the screen before drawing anything

beginScene() - this function is a stub for OpenGL and calls to directX begin scene function in DirectX implementation

Engine::Scene::getRenderableScene()->drawScene() - This functions actually draws all of the game objects in the renderable scene

endScene() - this function is similar to the beginScene(). It is a stub for OpenGL and contains directX end Scene function in DirectX implmentation.

showBuffer() - This function swaps the drawn objects from back buffer to the front buffer

Moreover, I also changed my implementation of Initialize() and ShutDown() similar to the Render function. My Initializa() and ShutDown() function looks like below image:

Second part of the assignment is to create platform independent shader code. The genearated code was not completely platform independent but it gave an idea how to write one. For this we included a platform specific file "shader.inc" which contains a wrapper conversion from GLSL to HLSL and vice versa depending on the current platform of the solution. Below is my vertex shader which have #include shader.inc.

I chose HLSL variable type name. There was no specific reason I preferred it - more likely it doesn't matter as shader.inc will convert it depending on the platform. But we should be careful, if we are using more than one variables in platform-independent code in the shader. It should be same throughout.

I also chose to write platform independent main function for the vertex shader. I used the way as John-Paul suggested - to create #define O_POSITION which has platform specific definition in the shader itself.

The interesting part of the assignment was to find a way so that BuildSystem will recognize the dependency of the "shader.inc" and build only if it presents. I created the dependency table for each asset and used it in BuildAsset lua file to check the dependency.

I read the dependency table for each asset in BuildAsset file and changed the signature of the function accordingly to accomodate the dependency table like below

Moreover, I also checked the existance of dependency along with the requirement that if the dependency is changed, builder should be able to build that specific asset.

This assignment was fairly easy and hardly took 2 hours of actual implementation. Moreover, I caught up in one bug which took more than an hour alone to fix it. Write-up almost took an hour. With almost four hours of work, I got the expected output.

Here is the [DirectX](https://www.dropbox.com/s/0rz51qkoni1xli7/Assignment_09_DirectX_Release.zip?dl=1) and [OpenGL](https://www.dropbox.com/s/50tb1eepsf959m2/Assignment_09_OpenGL_Release.zip?dl=1) link to the executable. Controls 'WASD' and Arrow Keys