---
layout: post
title:  "Continuous - C# and F# IDE for the iPad"
date:   2016-07-06 17:52:11 GMT
redirect_from:
  - /post/147003028753
  - /post/147003028753/continuous-c-and-f-ide-for-the-ipad
thumbnail: "http://continuous.codes/images/continuous_app.png"
---



Over the past six months I have been working on a new .NET IDE for the iPad, and today I am very pleased [to release it on the App Store](https://itunes.apple.com/us/app/continuous-.net-c-and-f-ide/id1095213378?ls=1&mt=8).

<img src="http://continuous.codes/images/continuous_app.png" alt="Continuous IDE on an iPad" />

[Continuous](http://continuous.codes) gives you the power of a traditional desktop .NET IDE - full C# 6 and F# 4 language support with semantic highlighting and code completion - while also featuring live code execution so you don't have to wait around for code to compile and run. Continuous works completely offline so you get super fast compiles and your code is secure.

Continuous gives you access to all of .NET's standard library, F#'s core library, all of Xamarin's iOS binding, and Xamarin.Forms. Access to all of these libraries means you won't be constrained by Continuous - you can write code exactly as you're used to.


### Real Work, on the iPad


I love the iPad but was still stuck having to lug around my laptop if I ever wanted to do "real work". Real work, in my world, means programming. There are indeed other IDEs for the iPad: there is the powerful [Pythonista](http://omz-software.com/pythonista/) app and the brilliant [Codea](http://twolivesleft.com/Codea/) app. But neither of those apps was able to help me in my job: writing iOS apps in C# and F#. I couldn't use my favorite languages on my favorite device and that unfortunately relegated my iPad to a play thing.

That realization produced this tweet last December:

> I resolve to use my iPad Pro for software development in 2016.— Frank A. Krueger (@praeclarum) January 1, 2016


<script async="" src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


Well it took me a bit of time, but I finally have it: a .NET IDE on the iPad (and phone too!).

But it's not "just an IDE". I didn't want it to simply be sufficient - I wanted it to be great. I also thought it was a nice time to push the state of the art in .NET IDEs a tad.

For ages compiled languages like C# and F# have forced a sequential development loop on programmers: the Code-Compile-Run-Test loop. We code something up, wait for it to compile, then wait for it to deploy and run, then we get to test it.

I hate waiting for compilation and deployment so I designed Continuous to minimize those steps. It does this by eagerly compiling your code - never waiting for you to tell it when to start. It runs your code as soon as those compiles complete successfully and displays the results of that execution right next to your code. Now you can focus on the code and the results of that code instead of being distracted by all the silly machinery of a compiler and IDE.

The benefits of making compilation and execution fast have surprised me. My iPad has become my favorite place to write apps now.

* The UI is visualized right next to the code that is building it.
* I am no longer constrained by designers with their static view of the world - the UI objects in Continuous are live and interactive.
* I can use real code files but still visualize objects out of them as if they were scripts.
* I can focus on building one screen of my app at a time and see the results without having to navigate from the first screen to see the screen I'm working on over and over.

I could argue that I'm a more efficient programmer thanks to these changes. Perhaps I am more productive. But the truth is, I'm just happier using Continuous. I play with GUIs more now, trying new ideas and tweaking things left and right. It's quite liberating and plain old fun to get nearly instant feedback on your work.

I hope you find these features as exciting as I do. Please [visit the website](http://continuous.codes) if you want more details on them, or throw caution to the wind and [buy Continuous on the App Store now](https://itunes.apple.com/us/app/continuous-.net-c-and-f-ide/id1095213378?ls=1&mt=8) to see them first-hand.


### Standing on the shoulders of giants


Continuous wouldn't be possible if it wasn't for .NET's great open source ecosystem. Continuous uses [Roslyn](https://github.com/dotnet/roslyn) for compiling C# and [FSharp.Compiler.Service](https://github.com/fsharp/FSharp.Compiler.Service) for compiling F#. Continuous also relies heavily on [Cecil](https://github.com/jbevain/cecil) (what problem can't be solved with Cecil?) Also, [Xamarin.Forms](https://github.com/xamarin/Xamarin.Forms) could only be included thanks to Xamarin open sourcing it.

And of course, none of this would be possible without mono and Xamarin.


### Colophon


I wrote Continuous in F# using Xamarin Studio. The code is more functional than object oriented and uses a redux style architecture. I don't think I could have built such a large app with its sophisticated requirements without F# at my side. Three years ago I wasn't sure how to write GUI apps in a functional language, now I question why I haven't always done things this way.
