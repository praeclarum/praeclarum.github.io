---
layout: post
title:  "Introducing NGraphics"
date:   2015-03-16 21:14:42 GMT
redirect_from:
  - /post/113814379658
  - /post/113814379658/introducing-ngraphics
thumbnail: "/images/tumblr/113814379658_0.png"
---



Many moons ago I created a cross platform library for rendering vector graphics called [CrossGraphics](https://github.com/praeclarum/CrossGraphics). It was good. It supported lots of platforms and was fast. Very fast in fact because it was designed for my app iCircuit which was already a CPU hog and had to run on 2010 era mobile hardware. To be fast, I played loosey goosey with cross platform accuracy (they *mostly* rendered the same) and features (who needs gradients?).

But the times, they are a changin’. My phone’s CPU is 64-bits and massive. Apple now supports GPU based rendering (finally that G stands for something other than Triangle). And, the fact is, though flat is in style, you really do need gradients.

![image]({{ "/images/tumblr/113814379658_0.png" | absolute_url }})

Today, I would like to announce [NGraphics](https://github.com/praeclarum/NGraphics). A cross platform PCL to satisfy your 2D vector drawing needs. It’s [available on nuget now](https://www.nuget.org/packages/NGraphics/)!

NGraphics is a big improvement over CrossGraphics. Here’s why I’m excited:

**Platforms!** NGraphics comes out of the box supporting iOS, OS X, Android, and Windows.

**Gradients!** It supports both the linear and radial variety with any number of color stops.

**Paths!** Crazy paths with arcs can now be used and abused.

**Images!** Images can be loaded from streams, created by rendering onto a canvas, or built up pixel by pixel.

**SVG Reading and Writing!** CrossGraphics was able to output SVG, but NGraphics can read 'em too.

**Retained Mode and Immediate Mode!** NGraphics supports both immediate mode rendering with canvases (DrawLine, FillPath, etc.) along with a retained mode graphics model (Path, Rectangle, Ellipse, ...). This makes caching and serialization easy.

**Unit Tests!** It turns out that getting all the platforms to behave themselves requires more than blind faith. NGraphics has a unit test suite that runs on all the platforms (4) and generates images to easily spot consistency errors.

**Editor!** I love to use code to draw things, but I hate waiting for the compiler (or worse, the IDE). So I wrote a little editor that lets you type C# code and get a live preview of the graphic you are creating. I love this thing...

![image]({{ "/images/tumblr/113814379658_1.png" | absolute_url }})

I hope you’ll check out NGraphics! It’s as simple as going to NuGet and searching for “NGraphics”. If you have the time, I would love bug reports, feature requests, and especially pull requests over at the [NGraphics project](https://github.com/praeclarum/NGraphics).
