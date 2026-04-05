---
layout: post
title: "Game engine coursework: Material builder"
date: 2015-03-08
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/materialbuilder"
---
## Material Builder

The intent of this assignment is to understand the what a material is and how we can use it to change the effects of an object in the graphics pipeline. Material is a common term to define the behavior of certain objects; for example wood, metal, gold are material. Our Material object is a wrapper object for such material to get the saimilar effect as that of the real material. Using this we can create different effect for the objects using the same effect for the object. The assignment can be thought of as split into two category - create a human redable material format and use it to create a binary file and load it runtime. Another part is to use that material data to change the effects of the object.

In this assignment, we were not asked to do the requireemnts in particular order and has been asked us to decide one which suits best for one. I chose to create first human, readable file and create the binary out of it. My human readable format for the material object look like as in below image.

I chose this format as it is pretty easy to understand just by looking the file. As one material will be using only one effect; works as key-value. On the other hand, material can have multiple uniforms for different shader, so I chose it to make as a table. If we see the individual entries for the uniform, "name" is the uniform name, shader is the type of shader for which uniform will be used. I additionally gave the data Type in cases if we want to have matrix types. As binary file format is concerend, I wrote like below:

As suggested, I created the typedef to define the type of the Handle as per the platform. Moreover, I also initalized it with some value rather than the garbage as it made it easy to debug.

Below is the one of the image of one of the binary file for the material generated for DirectX Release and OpenGL release mode respectively. There is subtle differnce between Win32 and x64 bianry files as the Handle size different for the platforms. It is a 8-byte (char*) in Direct3D(x64). While in openGL (win32), it is just an int and only takes 4-byte space. Thus the Direct3D binary files will be slightly larger than the one in win32 platform.

On the other hand, there is no differnce between release and debug mode for a particular platform pertaining for the size. There may be a different garbage values for uninitialized values for floats and handle. I didn't see this difference in my implementation as I was initializing it. However, I saw a differnce in padding value for debug and release mode - which doesn't matter as we usually don't care about it.

I felt, initially, to be easy but due to confusion I have to knid a implement everything thrice. Initially, I kept the uniform name in the uniform structure and implemented accordingly to the rest of the assignment. But after attending the class, I realized the way I was doing wasn't the requirement and I have to change everything again. When I finished the assignment, I realized that the objects behavior are not changing for the different material but same effects. This took a little time to fix this. The reason for this behavior was that I was using the shared effect and meshes due to which even if the material is different, it was not creating the new effect and was using the first handle value assigned to. To fix this, I redesigned the material structs and implemetation for the setting of the handles. - I think this was my mistake, I haven't read the assignment page in detail, rather I just got an overview and start implementing it. If I would have read it properly, I might not have to redesign this part.

Another interesting bug I faced in this assignemnt. While reading the binary file, I was doing the memcopy the entire array of Uniform to the array of the Uniform structure. It was only copying the first struct rather than the entire. It was really weird at first, but when I saw the size of the Uniform struct at runtime - it was different than the one in the binary file. The reason was the padding; when I was writing to the binary file, I was writing one by one rather than the complete structure for the uniform. But while at runtime, I was trying to copy the complete structure , which was not happening due to the diffrence in the size.

This assignment took almost 12-15 hours of mine to finish exactly as required and finally got the final result as below image.

Here is the [DirectX](https://www.dropbox.com/s/092lu0vrr3fvv1p/Assignment12_DirectX_Release.zip?dl=1) link to the executable. Controls 'WASD' for Camera Movement and Arrow Keys for Cube Movement.