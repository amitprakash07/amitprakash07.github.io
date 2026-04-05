---
layout: post
title: "Game engine coursework: Mesh builder — binary mesh assets"
date: 2014-12-14
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/meshbuilder"
---
## Mesh Builder - creating assets in Binary format

The purpose of this assignment was to understand the increased performance in the process of loading binary assets at runtime compared to loading of the human readable format. Earlier, we were using human readable formats for the assets, which are good for understanding but they are not good in loading at runtime. The reason for human readable files taking longer time is pretty simple - as the files are stored in ASCII characters, parsing them takes longer time. But if we use a binary format which is a format which machines read pretty fast, the loading will be quicker. Although, as of now, we can't see much larger difference in it as our mesh files are very small, but when the mesh files will be bigger, difference will be subtle. Another benefit of using the binary files is size - size of the binary files are very small caompared to the human readable format. As the shipped game will have binary files, thus the shipped game contain more meshes/digital content on one disk. We can see the difference of teh file size in below images.

Format of the binary file is very important. BInary files contains only bytes, so it is important to create the format simple as we need to read it back again. The number of elements for every structure, we writing to it, is important. The reason being if we don't know the count of the strucutre, we won't be able to cast/store the right data to its corresponding structure while loading it back. Earlier , I was thinking that as we know the size of the structure, we can get the count of the structure easily. It is okay to use this approach when we have single structure in file. But as we are storing multiple information - vertices as well as indices - there is no simpler way to get that count and to load them properly back. We can keep the count and data as per our convenience in the binary file but I chose one which was discussed in the class, as it was simple to understand and debug.

As my human readable asset file have five information, rather than four, I thought I should add five information to the binary file too. Although, I use four write functions to add these information, but there was no way, I was aware of, to read the five information with four extraction reads from the buffer, thus I used five read extractions. Earlier, My format of the bianry file, writing to it and reading from it was like below:

But as I am generating different files for different files, there is no use of winding information in the binary file as I can change the winding of the indices while creating the binary file itself. So no more need of writing this information, thus no more read. I understood this part when John-Paul asked me couple of cross questions upon my previous approach. Now, I changed my format, straightforward, like below.

Second part of the assignment was to create effect structure, similar to the mesh structure. I prefer, most of the time to use classes compared to the structures, thus I create the seffect class. In addition to, the effect class, I created an Effects static class which keeps all the effects created and can easily frees them once we are closing the application. I used the Effects class insterface functions to load the shaders which calls the platform dependent implmentation for them. Below is my code snippet for those.

This assignment was fairly easy and it took hardly 3 hours of mine in its implementation. However, I encountered with a runtime error with release - Direct3D_64 configuration, which took almost close to 3-3 and 1/2 hours of mine to debug and fix it. Moreover, I spent an hour for the write-up. With, cumulative, 6-7 hours of work, I got the same output for both platforms.

Here is my [DirectX](https://www.dropbox.com/s/ubr25jbj6abxxb8/Assignment06_DirectX_Release.zip?dl=1) and [OpenGL](https://www.dropbox.com/s/6o12ykxn1f80m8j/Assignment06_OPenGL_Release.zip?dl=1) executables.