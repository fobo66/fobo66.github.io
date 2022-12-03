---
title: "3D graphics guide for Android: OpenGL and related concepts"
published: true
---

Hello!

Some time ago I learned about demoscene in the BBS network in the early 90s. People have created amazing graphical images purely with Assembly code on a very limited hardware, and it was so great that even after all these years enthusiasts continue to create new demos and preserve old ones all over the world. I got inspired by the demoscene and decided to try something like this myself. I'm not familiar with Assembly (and, frankly, don't want to touch it), so I chose to create some demos on Android using OpenGL. It might not be as impressive, but at least I can use programming language I am familiar with (Kotlin) to set up some scaffold code for the actual OpenGL graphic code. Unfortunately, the info around computer graphics and OpenGL is quite inconsistent and scattered across different places, so I decided to summarize my learning experience in one article for future reference, so other people could benefit from this too.

In this article, we're going to walk through the main concepts behind computer graphics and OpenGL in context of Android. It's intended for learning, but you may need some basic knowledge about computer graphics, math and Android to make the most out of it.

## Basic graphics terms and concepts

This part is kinda optional if you already know how graphics works, I just wanted to summarize my knowledge and make sure I don't miss anything. It is basic definition of the fundamental terms and concepts.

Computers have a hard time creating visual data. They see image as a pile of binary data and don't by default understand how to put this chunk of bytes on the screen, so it's necessary to transform whatever you want to draw into appropriate format consumable by computer, so it can take it and draw it on the screen.

It was decided that the best approach to represent visual data on screen is using pixel – a single unit of data, like atom in the molecule. Each pixel has position and color. So it's left to figure out how exactly we can draw image on the screen using pixels.

There are 2 ways of doing it: raster and vector. Raster graphics means that the image is presented two-dimensional array of pixels (in the simplest case), and these pixels are merely structures that hold some info about the color of the point encoded in one of the many formats. For example, we can have 3 numbers from 0 to 255 that will represent hues of red, green and blue. This way, our pixel will be RGB-encoded. There are multiple possible ways to encode a color, but the RGB is the simplest.

If we need to draw some complex shapes on the screen, or produce some rapidly changing animated 3d object, raster drawing would be too hard. It will involve complex math to calculate shape's pixels and place them on the screen. And vector graphics allows us to do exactly that but without much hassle. We need to define shape with some params that we need, and the computer will do the math and draw the shape exactly how we define and on the fly. Vector images can be scaled with no quality loss, so if you put vector icon on the small WearOS watch or on the billboard, it will look sharp, while raster image will likely be a mess when scaled too much up or down.

Graphics involves a lot of math, and despite computers are good at math, they struggle to draw stuff with CPU only. It was enough in the early 90s to have only CPU to draw images, but the CPUs were quickly pushed to the limits. And not because they were less powerful than now, but because they were single-threaded. So people created separate processing units for graphics (GPUs) that can calculate a lot of math in parallel. It required them to change how the graphic data is actually drawn on screen. Modern GPUs contain thousands of cores with frequencies sometimes more than 1 GHz, so the can calculate billions of operations with ease, mostly thanks to this new approach.

## OpenGL

[OpenGL](https://www.opengl.org/) is a standard to describe this new approach in drawing graphic. It consists of the specification and programming language definition to work with this specification. It is maintained by Khronos Group and kept open source. I will shortly cover the internals of the OpenGL for the curious minds and introduce main concepts here.

## OpenGL on Android

## Next steps
