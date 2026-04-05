---
layout: post
title: "Game engine coursework: Index and vertex buffer (cross-platform)"
date: 2014-11-02
tags:
  - game-engine
  - graphics
  - coursework
source_wix: "https://amitprakash07.wixsite.com/home/meshcreation"
---
## Index/Vertex Buffer and Cross - Platform Implementation

This project was intended to understand the concept of Index and Vertex Buffer and its advantage of use. Vertex Buffer and Index Buffer are two buffers which many graphics API uses to render the scenes. Both of the buffers is just a memory space with different elemnents and referring to two different things. Vertex Buffer primarily contains the vertex location in world space. Depending on the implemnetation, we can add other information too to it. We, on the other hand used the vertx buffer to keep Vertices and Color of those vertices.

Usually, vertex buffer contains all the vertices information depending on the scene. In our case, we were drawing a square, which primarily requires two vertices. However, we are using primitive type as triangle - we need to draw two triangles ( 6 verticves) to create a square. But if we observe closely, while drawing the square using two traingles, both triangles have two common vertices. It looks we are passing redundant data o the vertex buffer, consequentially using more memory. Here comes the picture of Index buffer. Index buffer can store the indices of the triangles which need to be drawn and we can keep the indices in it in our required order (depending on the platform). With use of index buffer, vertex buffer just need to keep unique vertices information which reduces the size of the buffer drastically for biger scenes. In our case, as we are drawing a sqaure, vertex buffer will have four unique verttices. On the other hand Index buffer need to keep (6 indices - three for each triangle). But if we look at the big picture, with a larger scenes, we are reducing the size with a greater ratio.

Another part of the assignment is to solve a design issue. Our requirement is to create a structure, Mesh, which can be called to draw itself depending on its platform. The main goal was to separate out the draw implementation of different object, if any, in the structure itself. However, this draw implemnetation should also be carried to accomplish platform independence. With this approach, Graphics system only needs to call the draw method of the Mesh (or other specific objects- if any in the future). This makes the assignment interesting in a way how to separate interface, however, graphics system uses this shared interface to render by calling the draw method for the mesh. Below is my declaration for the Mesh structure, which contains members variables depending on the platform which is required for the platform dependent implmentation in draw() method. On the other hand Graphics system only needs to call the drawMesh() regardless of any platform.

Initially, Only problem which I encountered, I guess everyone, is how to call the Direct device to the interface implementation. I chose to create a class and made the device as its member variable and was using it by getter method from another places (Mesh implementaiton). But when I started implementing i got confused where exactly the implementation of 'vertexarray' and 'index array' should be - in the mesh implementation or in the grahics system itself. After thinking for a while, I chose to let it reside in the graphics system itself. The reason behind this is that, it's not always mesh needs a vertex array - in future we may need vertex array for other objects too. So we can accomplish this by just passing the size of the vertex structure to the graphics system and rest it can take care. This will help to avoid redundant code.

As per the implementation flow is concerned, I will try to brief it using the below image. My Mesh platform dependent implementation calls the Graphics system to get the device using its getter method. By looking closely to the below image, it's easy to say that OpenGL implementaion is not using anything from graphics system. The reason is OpenGL doesn't require any devices reference like DirectX. OpenGL wraps most of its implmentation in the API itself.

In the initialization of graphics system, I set the vertex array and index array to the Mesh structure member variables, IndexBuffer and VertexBuffer - which were used in the drawMesh Implementaion. Graphics system now only needs to call the drawMesh function without worrying about its platform dependent implementation.

The first part of the assignment didn't took too long, 30 mins. But the other part of the assignment requires design thinking in addition to the code refactoring and implementation of the Mesh structure. The second part almost took 8-9 hours in the implementation. With almost 9 hours of work, I finally got the expected output as below.

[Here is the Link to executable for the assignment](https://www.dropbox.com/s/jc24hfgi2d4mui1/Assignment03.zip?dl=1)
