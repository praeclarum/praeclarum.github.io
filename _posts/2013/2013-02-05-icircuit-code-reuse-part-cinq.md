---
layout: post
title:  "iCircuit Code Reuse, Part Cinq"
date:   2013-02-05 21:53:09 GMT
redirect_from:
  - /post/42378027611
  - /post/42378027611/icircuit-code-reuse-part-cinq
thumbnail: "/images/tumblr/42378027611_0.png"
---



I have toiled away with the new Windows 8 OS, the new Visual Studio 2012, and the new Office 13/365 to present you, dear reader, with this fine set of charts:

![image]({{ "/images/tumblr/42378027611_0.png" | absolute_url }})

(This post is the 5th in a series where I describe the code reuse of [iCircuit](http://icircuitapp.com) while porting it from platform to platform. Check out the [previous](http://praeclarum.org/post/15789866032/icircuit-code-reuse-part-trois)[posts](http://praeclarum.org/post/31799384896/icircuit-code-reuse-the-fourth-edition).)

Yesterday I completed the Metro, I mean Windows Store Modern App, version of iCircuit and achieved 85% code reuse from my other platforms! You should check it out, it's amazing to see Windows 8 actually do something useful! ;-)

[iCircuit for Windows 8](http://apps.microsoft.com/windows/en-US/app/icircuit/4041b312-408b-4bea-9c16-c49529230173)

To build this version, I used the work from the Windows Phone 7 port to get the basic app working in a XAML/C# solution. From there I "just" had to build up a modern desktop/tablet UI for the app. The initial port took about 1 day. Then I spent about 2 months refining the UI to feel good.

Thanks to the amazing .NET/Mono platform I was able to **reuse 39,000** lines of code and had to **write 6,700 lines of platform dependent code**. These figures represent a **code reuse of 85%**. This is on-par with all my other ports, and so I deem it a success.

That said, I'm a little disappointed that I had to write 6,700 lines. I was hoping for code reuse more inline with the OS X port of the app where I only had to write 4,000 loc. I blame WinRT's immaturity. You would be shocked to see some of the crazy bits of code I had to put in because the Win8 platform, while very rich, is also very generic and doesn't help you at all to build standard apps (document based, tools, etc.) That is to say, Cocoa is a very mature platform designed to make apps feature-rich and consistent while also making the developer's life easy. WinRT on the other hand gives you rectangles and a blog post that says "good luck".

Post Mortem

* **Microsoft did a great job of porting XAML graphics** over to this new brave DirectX world. All my Silverlight code worked out of the gate and had decent performance.
* The **XAML implementation is slow** as a snail slithering up a mountain during a rainstorm **when it comes to changing the scale of render transforms**. I can pan around with high FPS but the moment you zoom in or out, the app feels like we're back in the 80s. Now, this is probably not Microsoft's fault since the perf profiler says that I spend all that stalled time in video card drivers (Intel 3000 on my dev machine), but Microsoft should lay some tough love on these hardware peeps.
* WinRT introduces** yet another way to track multi-touch**. Now we have to wire up 5 Pointer events instead of a single Touch event from Silverlight. Check out [my implementation in CrossGraphics](https://github.com/praeclarum/CrossGraphics/blob/master/src/XamlCanvas.cs#L345) to see the horror unfold. CocoaTouch nailed multi-touch on their first go. This is Microsoft's 3rd attempt. Let's hope they're happy this time.
* Just when I thought I had completed the port, I realized that the app isn't functional on non-touch devices - there was a whole list of actions you just couldn't do. So I had to write keyboard and mouse code to make that all happen. Coming from iOS, this felt crazy indeed. Why am I putting keyboard shortcuts into a tablet app? Because **WinRT apps have to be just at home on the desktop as they need to be on tablets**.
* The **WinRT XAML implementation isn't able to render the visual tree to bitmaps** (WriteableBitmap does not have the Render method). Let me just add my voice to the 1,000,000s of other devs as we cry "Why Microsoft, why???" To export PNGs, I had to resort to deep dark black magic of DirectX and Direct2D. Simple question to all the devs on the XAML team: if I can write code to render a visual tree using DirectX that gets dumped into a WIC bitmap, why the hell can't you?
* **Testing the Share Charm is a real pita**. One little mistake and you will have to reboot Windows (yes, I said reboot Windows) to make sharing work again.
* The **Visual Studio performance analysis tools are wonderful**. They give nicely detailed reports in a UI that makes... well, some sense. It's still lightyears behind Instruments, but it's functional and enabled me to fix some hot spots in the code.
* The** file system security model is ludicrous and stupid** and dumb and confusing and stupid and annoying and a pain to use. I had to disable one of my favorite features of iCircuit (subcircuits) because I couldn't find a way to open a StorageFile using just a path. "Access Denied" all over the place.
* The** media system is a piece of crap compared to what we had in Silverlight** and WP7. I, again, had to disable major chunks of the app (microphone, speakers, and buzzer elements) because WInRT doesn't expose the media system to C#. There are "capture" devices but they incur a 500ms delay because you can't control buffer sizes. And playback, well, let's just say it has many problems too. I greatly miss Microsoft.Xna.Framework.Audio. (Oh, and while we're at it, fuck you MS for cancelling XNA, your only good API. I feel like I can swear now that we're way down in the list.)
* The **graphics system of WinRT is very fragile**. I want to do real-time 2D vector drawing. Direct2D is perfect for this. But WinRT puts all sorts of limitations on onscreen rendering, most notably: you can only have 1 DirectX swap chain (view) per window. That means I can't use Direct2D for rendering the scope which means the scope is slower than it needs to be. Dear Microsoft, go spend a few minutes and see how beautifully CocoaTouch and OpenGL work together on iOS. You might get inspired.
* The **App Store review process was crazy fast**. It took 5.5 hours for the app to be placed on the store. While I'm ecstatic that the review process was so fast, it does leave me wondering. I spent days struggle over the [UX guidelines](http://msdn.microsoft.com/en-us/library/windows/apps/hh465424) debating the right way to exposes this feature or that. In the end, it looks like it doesn't even matter. I wonder if they even looked at the app? It's big and complex. It contains a hand-written C compiler for God's sake. It has 5 UI views that are individually larger than most apps on the store. Ah well, it will be up to the users to tell me if the effort was worth it.
* This **WinRT version lacks a natively optimized solver**. iCircuit for iOS uses [Apple's Accelerate framework](http://developer.apple.com/library/ios/#documentation/Performance/Conceptual/vecLib/Reference/reference.html#//apple_ref/doc/uid/TP40002498) to do the heavy lifting in its solver to get wonderful performance that only gets better as the CPUs get wider and Apple's engineers optimize the library. Windows has none of this. I ran into many problems trying to get this C# app to use a WinRT C++ component but eventually gave up. It's too bad Microsoft doesn't care enough about high-performance computing to integrate useful numerical libraries.
* I need a better way to write cross-platform documentation.

Thanks for tuning in one more time! I hope you found this at least a tiny bit educational!

So which platform is next? I'm thinking of porting it to Windows 7, but will wait to see how sales do first. :-) In the mean time, [MonoTouch](http://xamarin.com/monotouch) and I need to spend some quality time together...

* [iCircuit for Windows 8](http://apps.microsoft.com/windows/en-US/app/icircuit/4041b312-408b-4bea-9c16-c49529230173)
* [iCircuit on iOS/OS X/Android/WP7](http://icircuitapp.com)
