---
layout: post
title: "Game engine coursework: Shader builder and effect builder"
date: 2015-01-11
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/shadereffectbuilder"
redirect_to_wix: true
---
## Shader Builder and Effect Builder

The purpose of this assignment is to create additional asset builders for loading the binary formatted shaders and effects at run time. In this assignments we were asked to create two additionals builders - Shader Builder and Effect Builder. Effect Builder is responsible to generate a binary format of an asset file containing the path of the shader file. On the other hand, Shader Builder is intended to generate a binary formatted of the compiled shader depending on the target platform.

I created the effect file format very simple, easy to understand, like bwlow. It just contains name to the type of the shader and its path relative to the BuildAssetDirectory. I just added the asset file and its associated builder in the AssetToBuild file.

I used the simpler way to write the information to the binary format for the effect file. I wrote the null terminated strings to the binary file. As it was only a matter of extra 1 byte on the overhead of processing it at loading. My binary file looks like. In the below image, its easy to find the '0x00' entries for the null character.

Writing this way makes it more easier for me to read it back again at runtime. As I store it as null terminated string, I directly store the first value from the buffer and then using the offset I stored the next string for the fragmentshader. Below is the snippet for the code.

Creating the effect builder was easy part of the assignment; on the other hand creating the shader builder was a interesting part. We were given two options to implement it. Either ceate two builder - VerteXBUilder and FragmentBuilder- and associate relevant file in AssetsToBuild file with its builder too. This was easy way but there was a lot redundance of the code. Another suggested way was to push an optional argument to the ShaderBuilder somehow so that it can compile the shader according to its type. I chose to create the optional argument way, as it gives opportunity to scale the application up incases, when we have other types of shader too.

I added an extra piece of information to the AssetsToBUild.lua file for ShaderBuilder tool as like below. Moreover, I have to process the optional argument in the Lua file, I made some changes in the BuildAssets.Lua file too.

To accomodate this change to push this extra argument, I made the changes in the BuildAssets functions in the BuildAsset.lua file as like below. Now, I am checking for the additional inforamation in the passed asset and creating different tables for different cases.

Moreover, I changed the signature of the BuildAsset function to process the newly created table, like below.

And, I am pushing the optional argument by concatenating the arguments with optional arguments - like below

Another intuitive thing about the shader builer is the use of #define EAE6320_GRAPHICS_AREDEBUGSHADERSENABLED. This macro is responsible for generating the optimized (small size) binary file when the solution configuration was set to RELEASE mode. This could have also been accomplished using _DEBUG pre-processor but the benefit of using this way is to have this special behavior apart from the default behavior of the _DEBUG or RELAESE configuration. Using this way, we can enable optimization feature for any configuration by enabling this macro even in some custom configuration. Apart from the way of toggling this feature, there is subtle difference on the size of the binary files. This size difference is due to the way it is build in each configuration. Moreover, build process with use of this macro is differennt for both platforms - DirectX and OpenGL.

In DirectX, these macro diables the optimization feature so that it will contain some information about the shader which will be helpful be while debugging the compiled shader code. We can easily see the shader code in the unoptimized version in the below image - i_posiiton, o_color etc.

###### DirectX Compiled Vertex Shader with EAE6320_GRAPHICS_AREDEBUGSHADERSENABLED

In the optimized version of DirectX compiled shader, we can see it only contains the compiled binary information and nothing else; which is good for the build meant for shipped as it will be faster to us.

###### DirectX Compiled Vertex Shader with EAE6320_GRAPHICS_AREDEBUGSHADERSENABLED disabled

On the other hand, these macro setting works in a different way for OpenGL. If it's enable it generates the code with comments (it just splits the GLSL code from the combined shader file). We can see the comments in below image along with the code. If the macro is disabled it doesn't generates any comment - thus the file size is smaller to process at runtime.

###### OpenGL Compiled Vertex Shader with EAE6320_GRAPHICS_AREDEBUGSHADERSENABLED - contains comments

###### OpenGL Compiled Vertex Shader with EAE6320_GRAPHICS_AREDEBUGSHADERSENABLED disabled - doesn't contains comments

With the mid-term review, John-paul pointed out some errors in my code which I fixed in this assignment. First was that, my application process was keep on running even after window is closed. I fixed this by little refactoring win my windowing system. Another concern was the double inclusion of functions in the static libraries. This I removed from this build by adding additional command line option in the project to ignore this linking warning. Another concern was that my solution was building BuildAsset everytime, so I changed the dependency a bit so that it won't get build every time when we start the application.

Moreover,John-Paul pointed out a bug in my code, The bug was was that shader code was loading every time there was a draw call. Earlier, I was loading the shader while setting the shader due to which it was every time loading it - which was bad performance wise. I changed bit of my code so that it will only load the shader when effect is getting created. This was just small refactoring of the code but the efect was noticable, object was moving without jitter in the game.

As told by John-Paul, this assignment will take 6-10 hrs of work, It almost 8-9 hours of mine. The assignment wasn't that tough. Pushing the optional argument to the shader builder took more time. Implementation wise, the assignment took 3-4 hours, while debugging and checking everything is working or not took almost 1 hours. I caught up in one issue, when my openGL build wasn't working, it took almost 1 and half hour to find the issue, but i was unable to find it. Upon answer of John-Paul, it hardly took me half hour to figure out what I did wrong and to fix it. Moreover, fixing the previous bugs took like 2-3 hours, that doesn't counts for this assignment, rather should be earlier assignments. Write-up took almost an hour for me.

As expected, I got the right outputs as below.

Here is the [DirectX](https://www.dropbox.com/s/865h4zs17z1jf1g/Assignment_8_DirectX_Release.zip?dl=1) and [OpenGL](https://www.dropbox.com/s/3jsbi8y65ph4u38/Assignment_08_OpenGL_Release.zip?dl=1) link to the executable. Controls 'WASD' and Arrow Keys
