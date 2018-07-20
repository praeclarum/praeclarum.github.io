---
layout: post
title:  "Interfaces + caches = Cross Platform"
date:   2010-11-09 00:19:31 GMT
redirect_from:
  - /post/1520024382
  - /post/1520024382/interfaces-caches-cross-platform
---



iCircuit is a mobile app that lets you experiment with circuits on your iOS devices. Well, soon, it will also let you do it on your WinPhone7.

It was implemented using the innovative and very rewarding [MonoTouch](http://monotouch.net) system. MonoTouch lets you write native iOS apps in C# with full access to .NET. MonoTouch + the iPhone made programming fun again. It was the best decision I've made in a long time.

On the evening of Jul 21, I was frantic waiting for Apple to approve my baby, and somewhat curious as to how much effort it would take to port the app to Silverlight.

The engine ported immediately, without change, since it was written in pure .NET and only communicated with the UI through interfaces. (Well, properly said, the UI only communicated with the engine through interfaces.)

After that, I only had to port the UI which was broken into two nice big chunks: The graphical circuit editor and the "chrome" which handled file management and all that boring stuff that makes apps "apps".

The graphical UI was again coded against interfaces only. So to port it to a new platform, I would merely (haha, "merely") have to implement those interfaces. Because drawing on the iPhone is immediate mode (DrawCircle(); DrawLine(); ...) I designed that interface along those lines. Nearly every graphic interface in existence is immediate mode: iOS, CoreGraphics, Windows GDI, GDI+, OpenGL, DirectX, whatever you call the rendering system on Android, the Canvas element in HTML, ... the list goes on and on. But there are two huge exceptions: Silverlight and WPF!

Here I had a perfectly abstracted graphics driver that fit every platform in the world except Microsoft's new platform. Yawn. So I never got around to porting the app.

But on July 21, again, I was frantic and needed to code something. So I remembered the golden rule of programming: caches fix everything. I went about writing an implementation of my immediate mode interface that translated all those Draw*() calls to the retained mode of Silverlight (line = new Line(), Add(line)). What followed was the following check ins:

``

```csharp
changeset:   305:ea9de0ac4a9e
user:        Frank A. Krueger 
date:        Wed Jul 21 17:49:19 2010 -0700
summary:     OMFG it's actually working

changeset:   304:7c5b310e602c
user:        Frank A. Krueger 
date:        Wed Jul 21 17:27:11 2010 -0700
summary:     Fixed bug in my crazy algo, now it works!!!

changeset:   303:fe1bd8e4a3be
user:        Frank A. Krueger 
date:        Wed Jul 21 17:17:08 2010 -0700
summary:     Work toward making the silverlight circuit render. Amazed that my idea actually seems to be working.

changeset:   301:0f223a67e40d
user:        Frank A. Krueger 
date:        Wed Jul 21 15:04:02 2010 -0700
summary:     Began the Silverlight port. CircuitLib is compiling. Yeah. Now I need to reimplement the actual UI :-(
```


``

In a little less than 3 hours I had it working. God I love caches and interfaces.

Making it work on the phone was just a matter of setting up the project in Visual Studio and recreating the chrome (something I'm still working on today).

Today, Nov 8, I got to find out if my method worked - and oh boy did it! The phone is chugging along right now rendering circuits.

In conclusion: I love .NET. I love MonoTouch. My WinPhone7 is sweet. It's a great time to be a developer!
