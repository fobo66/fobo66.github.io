---
title: "3D graphics guide for Android: OpenGL and related concepts"
published: true
---

Hello!

Some time ago I learned about demoscene in the BBS network in the early 90s. People have created amazing graphical images purely with Assembly code on a very limited hardware, and it was so great that even after all these years enthusiasts continue to create new demos and preserve old ones all over the world. I got inspired by the demoscene and decided to try something like this myself. I'm not familiar with Assembly (and, frankly, don't want to touch it), so I chose to create some demos on Android using OpenGL. It might not be as impressive, but at least I can use programming language I am familiar with (Kotlin) to set up some scaffold code for the actual OpenGL graphic code. Unfortunately, the info around computer graphics and OpenGL is quite inconsistent and scattered across different places, so I decided to summarize my learning experience in one article for future reference, so other people could benefit from this too. It may seem scattered, but it's how I understood stuff.

In this article, we're going to walk through the main concepts behind computer graphics and OpenGL in context of Android. It's intended for learning, but you may need some basic knowledge about computer graphics, math and Android to make the most out of it.

## Basic graphics terms and concepts

This part is kinda optional if you already know how graphics works, I just wanted to summarize my knowledge and make sure I don't miss anything. It is basic definition of the fundamental terms and concepts.

Computers have a hard time creating visual data. They see image as a pile of binary data and don't by default understand how to put this chunk of bytes on the screen, so it's necessary to transform whatever you want to draw into appropriate format consumable by computer, so it can take it and draw it on the screen.

It was decided that the best approach to represent visual data on screen is using pixel – a single unit of data, like atom in the molecule. Each pixel has position and color. So it's left to figure out how exactly we can draw image on the screen using pixels.

There are 2 ways of doing it: raster and vector. Raster graphics means that the image is presented two-dimensional array of pixels (in the simplest case), and these pixels are merely structures that hold some info about the color of the point encoded in one of the many formats. For example, we can have 3 numbers from 0 to 255 that will represent hues of red, green and blue. This way, our pixel will be RGB-encoded. There are multiple possible ways to encode a color, but the RGB is the simplest.

If we need to draw some complex shapes on the screen, or produce some rapidly changing animated 3d object, raster drawing would be too hard. It will involve complex math to calculate shape's pixels and place them on the screen. And vector graphics allows us to do exactly that but without much hassle. We need to define shape with some params that we need, and the computer will do the math and draw the shape exactly how we define and on the fly. Vector images can be scaled with no quality loss, so if you put vector icon on the small WearOS watch or on the billboard, it will look sharp, while raster image will likely be a mess when scaled too much up or down.

Graphics involves a lot of math, and despite computers are good at math, they struggle to draw stuff with CPU only. It was enough in the early 90s to have only CPU in the personal computer to draw images, but the CPUs were quickly pushed to the limits. And not because they were less powerful than now, but because they were single-threaded. So people created separate processing units for graphics (GPUs) that can calculate a lot of math in parallel. Modern GPUs contain thousands of cores with frequencies sometimes more than 1 GHz, so the can calculate billions of operations with ease.

## OpenGL

[OpenGL](https://www.opengl.org/) is a standard to describe this new approach in drawing graphic. It consists of the specification and the C framework aligned with this specification. It is maintained by Khronos Group and kept open source. I will shortly cover the internals of the OpenGL for the curious minds and introduce main concepts here.

In general, you don't need to use any framework to draw stuff on screen. OS and its components usually provide enough handles for the developers, so they can draw whatever they want. But you will need to write a shit ton of boilerplate code just to set things up and make the environment ready for drawing. It's like manufacturing your own custom paper and pencil out of raw materials (wood, cotton, etc.) and with the provided tools, just to draw some doodles. It is suboptimal at least. But at the time of OpenGL creation, there was no other choice.

Luckily, people realized that and started to think about generic framework for graphics that will contain the complexity of setting up stuff for each platform and offer convenient tool set for drawing basic building blocks of the image, so it can be applied for any use case.

OpenGL draws stuff pixel by pixel, and for each pixel we can provide a script with the logic necessary to draw this pixel. These scripts called shaders and should be written in the special language called [GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language). It is similar to C, but with some extra default stuff defined in the framework. Shaders can accept input data, which allows us to draw something that depends on a state, e.g. on the cursor movement or time.

There are 2 types of shaders in OpenGL: fragment shader and vertex shader. Fragment shader produces color data and similar visual stuff for the pixel, and it's enough to use just them for 2D graphics. Vertex is a position in three-dimensional space, so vertex shaders are needed to draw 3D objects on the screen, because they can get the position of the vertex of the model in the 3D space and convert it to the 2D space. Since our screens are mostly flat, it's necessary to have some way to project 3D objects on flat surface.

You will need to write most of the code for the shader yourself, but there are a lot of helper functions in the standard library implicitly included in the shader for you to use.

## OpenGL on Android

## AGSL

## Next steps
