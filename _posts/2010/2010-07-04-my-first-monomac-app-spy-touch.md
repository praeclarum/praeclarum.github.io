---
layout: post
title:  "My First MonoMac App - Spy Touch"
date:   2010-07-04 19:18:00 GMT
redirect_from:
  - /post/770071583
  - /post/770071583/my-first-monomac-app-spy-touch
thumbnail: "/images/tumblr/770071583_0.png"
tags: announcement
---



The Mono team has recently given the world [MonoMac](http://tirania.org/blog/archive/2010/Apr-19.html), a framework that allows C# programmers to write native OS X (Cocoa) apps. This is wonderful news for those who simultaneously love OS X and C#.

I decided to take her for a spin by writing a little (very little) app that will assist me in my app development.

Windows developers have an app called [Spy++](http://msdn.microsoft.com/en-us/library/aa242713(v=VS.60).aspx). I grew up with this little beauty as I was learning Win32 programming. It was invaluable for learning the structure of GUIs and discovering the little tricks other programmers used to make their GUIs work the way they did. Later, we got [Managed Spy](http://msdn.microsoft.com/en-us/magazine/cc163617.aspx) for WinForms and [Snoop](http://snoopwpf.codeplex.com/) for WPF.

We don't have anything like these tools for [MonoTouch](http://monotouch.net/). Well, now we do! (And by like, I mean: has 1/100th the features.)

![image]({{ "/images/tumblr/770071583_0.png" | absolute_url }})

This little app (can I say little enough?) displays the active visual tree of your app while it is running. As you navigate the visual tree, the active view gets highlighted in a little visualizer on the right hand side.

To use this tool, you just need to install a little stub client in your app. Include the file SpyTouch.cs, and put the line **SpyTouch.SpyTouch.Run ();** somewhere in your startup code. Here is a [package containing Spy Touch, the stub, and its source code](http://dl.dropbox.com/u/6908183/SpyTouch.zip).

Now you can monitor the visual tree of the app. It refreshes automatically every 2 seconds.

The usefulness of this tool isn't very high because it is just a proof of concept of tools to assist me in the design of GUIs for iOS apps. Now that I have this tool, I am very excited about future possibilities in constructing UI designers.
