---
layout: post
title:  "My Great Wumpus Hunt"
date:   2011-06-18 19:53:27 GMT
redirect_from:
  - /post/6663989742
  - /post/6663989742/my-great-wumpus-hunt
---



There is a bug in [iCircuit](http://icircuitapp.com) that is tarting to feel like my [white whale](http://en.wikipedia.org/wiki/Moby-Dick), my [wumpus](http://en.wikipedia.org/wiki/Wumpus). I know all about the bug: I know why it's happening, I know all its symptoms, and I know at least one way to squish it.

"So what's the problem?" you might legitimately ask. The problem is that the only fix, that I know for certain, creates a very non-Apple-esque feel to my app, and that is preventing me from implementing the fix. Apple provides the wonderful [UIScrollView](http://developer.apple.com/library/ios/#documentation/UIKit/Reference/UIScrollView_Class/Reference/UIScrollView.html) which, among other things provides momentum scrolling and very smooth animation. If I implement my own gestures, all my problems are solved except I miss out on the benefits of UIScrollView. End result for the user? The app feels "laggy".

I have tried 4 times to implement different strategies to fix this bug while maintaining the Apple-feel factor. I have failed 4 times. 

OK, enough generalizations, thanks for sitting through that. Here is the bug:

iCircuit uses a [UIScrollView](http://developer.apple.com/library/ios/#documentation/UIKit/Reference/UIScrollView_Class/Reference/UIScrollView.html) that contains 3 large (usually mostly off-screen) UIViews that componsite together to show you the circuit. This (amazingly) works fine thanks to the fantastic engineers at Apple. But I consider that implementation to be a bug because of these symptoms:

**Symptom 1: Everything is all blurry!**

![image]({{ "/assets/tumblr/6663989742_0.png" | absolute_url }})

When you zoom in to the circuit, everything becomes blurry. This is due to the simple nature of UIScrollView and the fact that I don't change the view hierarchy at all after a zoom operation. This is the default behavior of UIScrollView and drives me absolutely batty.

[Apple has a way to fix this](http://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/UIScrollView_pg/ZoomZoom/ZoomZoom.html#//apple_ref/doc/uid/TP40008179-CH102-SW9)! All you have to do is switch your code to be thread safe, stop using UIGraphics, and implement [CATiledLayer](http://developer.apple.com/library/ios/#documentation/GraphicsImaging/Reference/CATiledLayer_class/Introduction/Introduction.html) in the content view of the UIScrollView. Easy huh? It actually is, and works surprisingly well. When you zoom, everything gets redrawn at screen resolution.

You've seen this beauty before in apps like Safari. When you first zoom in, everything is blurry and then (pretty quickly) everything gets redrawn at the new native resolution. Another +1 to Apple engineers.

Great! But it doesn't work for iCircuit. The main UI of the app is animated at about 7 FPS (it adaptively changes the frame rate to keep your battery from dying) and this ends up being too much work for the CATiledLayer. The screen becomes a mish-mash of partially updated regions and smooth animation is out the door. This is an appropriate emoticon for my mood when discovering this: :-(

**Symptom 2: Way too much Memory**

A lot of users of iCircuit run into one limitation before all others: I limit circuit sizes to 2048x2048 pixels. This size limitation is in place for one and only one reason, it's because the app runs out of memory if you increase it.

Ugh. I feel like I'm 16 again implementing code in the most naive fashion just hoping it works. It's ridiculous to tie the logical dimensions of a circuit to an OS-tracked-super-heavy-resource. If the circuit is 2048px across, I create 3 UIViews that are also 2048px across. iOS can handle this, but what if you want a circuit that's 10 kpx wide? The OS just doesn't have enough memory (nor CPU) to render UIViews of that size at the framerate that I want.

So if I know this is the wrong thing to do, then why do I do it? Becuase it makes UIScrollView happy.

**Symptom 3: Retina Display is Sloooow**

Memory is one thing, but on iOS you get all the memory you ask for until the app is shut down. It's a good deal. CPU, on the other hand, well, there's only one of those and it has a lot of work to do. I'm already asking it to do a lot in solving the circuit equation in the background. Asking it to draw 2048x2048px images at 7 FPS takes about 10% of the CPU on iPhone 3GS and iPad 1.

Retina display doubles pixel counts. So my 2048x2048 circuit becomes 4096x4096px. This is an increase to memory, and worse, a burden to the CPU to draw lots of anti-aliased lines. It turns out that the iPhone 4 is not capable of this. Quickly, it falls to 3 FPS (lower limit) and the whole user experience is degrated.

Alas, this is the current state of iCircuit 1.2. I surprisingly don't get a lot of emails about this since the app is really meant for the iPad, but it breaks my programmer heart. Again, I feel like a 16 year-old punk.

**Solution 1: Wherein I try to fix it 4 times and fail every time**

This post is really just an opportunity for me to vent. I hate this bug. I hate that I haven't been able to fix it. I have ideas of how to make it better, but I haven't been able to code any of those ideas to my satisfaction.

In a nutshell, solving the problem involves keeping 1 huge UIView for the UIScrollView to deal with. Within that view, I will place a subview that draws only that part of the circuit that the user is viewing.

Sounds simple, right? A two-hour hack, right? The devil is in the details in this one, but I still have high hopes. I lied to you before: this blog post is actually a rallying call for me. I'm about to implement this solution for the fifth time. I really want it to work, wish me luck.

**Solution 2: Ditch UIKit/Core Graphics/Core Animation**

I love the Apple platform. After years of writing Windows apps where Microsoft said "here's a push button and a window object, good luck with the rest", it's a real refresher to work with powerful APIs that are geared toward making the programmer's life (and thusly, the end user's experience) better.

But if it doesn't work it doesn't work. The Apple platform has one last escape hatch when dealing with graphics-heavy apps: fall back to OpenGL. I know from past experiences that this will solve the majority of my problems:

1. Drawing will be fast because OpenGL is a monster.
2. This will free up the UI thread which means I can deal with the touch events. I won't be able to match 100% the smoothness of UIScrollView, but I can get close.

There are, of course, two hurdles preventing me from taking this path:

1. OpenGL isn't great at anti-aliasing. 1px lines look like they're from 1990 and my sould dies a little death. Ironically, this isn't an issue on retina displays since the DPI is so ridiculously high. But on chunky-pixel devices, this makes the app look amateurish at best. (Actually it makes it look like all those Windows circuit editors that I'm trying so hard not to be).
2. Fonts. If you've ever programmed OpenGL, you know what I mean. Fonts suck. They're blurry, they take effort to manage, and they're blurry.

**THE END**

Enough ranting and time to start coding. Please do wish me luck as I hunt and kill this monster. And, if, like times previous, the Wumpus should eat me for dinner, remember, it was all in the name of making an app worthy of the hardware.

