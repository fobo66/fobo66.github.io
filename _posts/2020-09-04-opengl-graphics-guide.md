---
title: "3D graphics guide for Android: OpenGL and related concepts"
published: true
---

Hello!

Some time ago I learned about demoscene in the BBS network in the early 90s. People have created amazing graphical images purely with Assembly code on a very limited hardware, and it was so great that even after all these years enthusiasts continue to create new demos and preserve old ones all over the world. I got inspired by the demoscene and decided to try something like this myself. I'm not familiar with Assembly (and, frankly, don't want to touch it), so I chose to create some demos on Android using OpenGL. It might not be as impressive, but at least I can use programming language I am familiar with (Kotlin) to setup some scaffold code for the actual OpenGL graphic code. Unfortunately, the info around computer graphics and OpenGL is quite inconsistent and scattered across different places, so I decided to summarize my learning experience in one article for future reference, so other people could benefit from this too.

In this article, we're going to walk through the main concepts behind computer graphics and OpenGL in context of Android. It's intended for learning, but you may need some basic knowledge about computer graphics, math and Android to make the most out of it.

## Basic graphics terms

Computers have a hard time creating visual data. They see image as a pile of binary data and don't by default understand how to put this chunk of bytes on the screen, so it's necessary to transform whatever you want to draw into appropriate format consumable by computer, so it can take it and draw it on the screen.

There are 2 ways of doing it: raster and vector. Raster graphics means that the image is presented two-dimensional array of pixels (in the simplest case), and these pixels are merely structures that hold some info about the color of the point encoded in one of the many formats. For example we can have 3 numbers from 0 to 255 that will represent hues of red, green and blue. This way, our pixel will be RGB-encoded. There are multiple possible ways to encode a color, but the RGB is the simplest.

If we need to draw some complex shapes on the screen, or produce some rapidly changing animated 3d object, raster drawing would be too hard. It will involve complex math to calculate shape's pixels and place them on the screen. And vector graphics allows us to do exactly that but without much hassle. We need to define shape with some params that we need, and the computer will do the math and draw the shape exactly how we define and on the fly. Vector images can be scaled with no quality loss, so if you put vector icon on the small WearOS watch or on the billboard, it will look good and sharp, while raster image will likely be a mess in this case.

## OpenGL

## OpenGL on Android

## Next steps
