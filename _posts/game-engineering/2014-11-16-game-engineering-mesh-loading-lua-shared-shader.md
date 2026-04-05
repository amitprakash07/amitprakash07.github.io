---
layout: post
title: "Game engineering: Mesh loading using Lua and shared shader"
date: 2014-11-16
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/meshfromusinglua"
---
## Mesh Loading Using Lua and shared Shader

This project was intended to achieve two things - how to use shared shaders and how to use load a mesh structure using Lua. The first part of the assignment was, comparatively, easy to the second part. The intent was to combine the platform dependent separate shader files in one file. Using #if defined for platform specific shader code, we can use single file to load different platform shader code. As part of loading shader is considered, we can specify the shader section and macro definition while loading the shader code, either for Open_GL or DirectX platform.

I chose to name my shared shader file as "standard.vshd' and 'standard.fshd'. The reason for adopting this naming convention is that there are different types of shader implementation apart from the programmable shader of the graphics pipeline. Using this convention, the file extension can refer to the programmable graphics pipeline shaders, while on the other hand filename can infer to the type of shader or its feature - deferred shading, MSAA features etc.

As part of the assignment, I haven't done any improvement to the shader file or shader loading code. But I have some improvement idea. As in the shader loading code, the openGL and DirectX can load any code for the shader from the sections of the file which may be defined by programmer using shader macros.

These macros guides OpenGL and DirectX graphics API to look for their corresponding shaders. We can use this feature to load any type of programmable shader/shader features and separate them using #defines in the a single file. Another improvement, I could think of toggling the features in the shader code by using the #defines in the cpp files like below.

const D3DXMACRO vertexShaderMacro[] =

#ifdef STANDARD { "PLATFORM_D3D", "1" }, {NULL,NULL}

#elif DEFERRED

{"DEFFERED_SHADER_MAIN","2"},

{NULL,NULL}

#elif MSAA

{"MSAA_MAIN", "3"}

{NULL, NULL} };

The second part of the assignment was to create a human readable format for the mesh and load it using Lua. The reason to have the file structure as human readable is very important. Sometimes, the use of mesh files structure is not only limited to the automted program but also to the artists and even to programmers. This gives the flexibility to the artists, especially, to understand and change the structure to see the effect on the scene too - without understanding behind the scene code. Another best part of the file being human readable is that it can be understood by anyone by reading the descriptions too.

I chose the mesh format as below picture and file name to be "Mesh.lua". I used a strightforward thought for using the file name, as it was used for describing the mesh stucture and was described in lua, so 'Mesh.lua' make more sense to me. As far as layout of the file is concerned, earlier, I used dictionary to define my vertices like v0 = {index = 0, position={..,...}}. But after reading JP's conjecture about using array and dictionary, I decided to change the structure. Moreover, I used this approach for another reason - we can add as many vertices as we want and traverse through each of the vertices in the C++ code rather than accesing as like dictionary key/value pairs.

Another thing I chose not to consider default was 'winding' setting. Giving that setting in file gives flexibility to change the order from the file itself (some exporter may not use 'right' winding) and in the cpp code,i was taking care of the changing the order of the indices according to graphics platform.

Another part of the requirement is to discuss about the color channels in the vertex structure. I chose to keep it as four - [R, G, B, A] - the reason was straightforward, as almost all applications uses these four channels to represent color. Although aplha channel, in our case - until now is 1, but may be in upcoming assignments we may need to change that for different materials.

First part of the assignment was very easy and with detailed description about implementation, it was easy to implement and it took hardly a half an hour for me to finish the first part. On the other hand, the second part of the assignment was quite tedious and time consuming for me as I caught up in couple of issues while implementing. Although it was tedious to parse the lua file but it helped me to understand the working of lua stack while using it along C/C++. With almost 14-15 hours of work, I finally finished the assignment and I got the output as below.

Here is the [link to the executable.](https://www.dropbox.com/s/m6ah2tfe2saziww/Assignment04.zip.zip?dl=1)
