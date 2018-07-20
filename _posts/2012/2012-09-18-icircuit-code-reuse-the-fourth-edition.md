---
layout: post
title:  "iCircuit Code Reuse, the Fourth Edition"
date:   2012-09-18 16:06:00 GMT
redirect_from:
  - /post/31799384896
  - /post/31799384896/icircuit-code-reuse-the-fourth-edition
---



Hell has indeed frozen over and I have released the [Android version of iCircuit](https://play.google.com/store/apps/details?id=com.kruegersystems.circuitdroid)! [iCircuit](http://icircuitapp.com) is now on 4 platforms!

I wrote the app using [Xamarin's Mono for Android](http://xamarin.com/monoforandroid) so that I could reuse large amounts of code that I have painstakingly written and debugged on other platforms.

That means it's time to run that [code reuse script](https://gist.github.com/1608597) and find out how I did as a developer. How much of the Android code base had to be written anew and how much was I able to reuse from the other platforms?

Here is a visualization of the code reuse. The percentage in green is the amount of code reused while the code in red is code I had to write for each platform.

![image]({{ "/assets/tumblr/31799384896_0.png" | absolute_url }})

We can see that Android was only ("only", haha) an 81% code reuse - I had to write nearly 7,000 lines of code to port iCircuit to it. This is great, but I was aiming for closer to the 90% mark as the Mac was able to achieve.

I can chalk this difference up to two new additions to the code base:

1. Two new UI controls have been added that are unique to Android. The first is a "mini-scope" that is shown persistently on the screen. The other is a new dial that makes it much easier to input numbers on a multi-touch screen. The dial will eventually make it into the other platforms (iCircuit 1.5).
2. Two new graphics drivers. The first uses [Skia (Android.Graphics)](http://en.wikipedia.org/wiki/Skia_Graphics_Engine) to render scenes, and the other uses OpenGL. The Skia driver is used to render thumbnails, draw the scope, and other UI elements, while the OpenGL driver renders the main editor (very quickly).

Code reuse aside, I'm very excited to have this released (it's taken me awhile), and hope all the Android engineers out there give the app a try!

(This is a followup to the post [iCircuit Code Reuse Part Trois](http://praeclarum.org/post/15789866032/icircuit-code-reuse-part-trois). More details on my methodology can be found there.)
