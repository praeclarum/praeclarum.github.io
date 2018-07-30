---
layout: post
title:  "Calca OS X Now Available & Code Reuse"
date:   2013-07-15 18:37:37 GMT
redirect_from:
  - /post/55529252776
  - /post/55529252776/calca-os-x-now-available-code-reuse
thumbnail: "/images/tumblr/55529252776_0.png"
---



[Calca](http://calca.io) is my latest app - a text editor and computer algebra system happily married together. The reaction to Calca has been overwhelmingly positive. I am very excited that people are finding it useful. Make sure to leave reviews and tell your friends. :-)

[Anyway](http://marco.org), I'm please to announce today that the [OS X version of Calca is available](https://itunes.apple.com/us/app/calca/id635758264?ls=1&mt=12). In this form, Calca really shines. It's the same powerful engine as before, but it's much easier to manipulate text documents on Mac and it can handle much larger documents.

I was even able to put in some convenience features from IDEs: matching parenthesis highlighting, matching identifier highlighting, and indent and commenting keys. The app is also iCloud enabled so all your Calca for iOS work is available. [I hope you enjoy it](http://calca.io)!

Following the tradition of iCircuit, I want to take a few minutes to report how much code reuse I was able to attain between the iOS version and the Mac version. To do so, I simply ran the [Code Share Measurement Script](https://gist.github.com/praeclarum/1608597). It produced the following raw data:

```csharp
app    t	u	s	u%	s%
Mac	12253	916	11337	7.48 %	92.52 %
iOS	14227	2890	11337	20.31 %	79.69 %
```


We can see that Calca's engine - the shared code - is 11,337 lines of code. One could say it's an elite engine.

Here is a picture showing that same data. The code in green is shared, while the code in red had to be written specifically for that platform.

![image]({{ "/images/tumblr/55529252776_0.png" | absolute_url }})

This is showing that the UI code of the iOS version is nearly 3,000 lines of code compared to 1,000 lines for the Mac version. The iOS version's UI is 3 times as big as the Mac's.

Why is the iOS code so much bigger than the Mac's? One answer: file management.

iOS human interface guidelines ask developers to minimize the use of the file system in normal interactions with our apps. To that end, there are no UI controls specifically designed for file management. There are no directory browsers, no file open or save dialog boxes, no UI to simply rename files. All of this has to be coded per application by the developer.

On iOS, Apple stands over your shoulder commenting "look how much effort it takes to write a document based application, are you sure you want to do it this way?"

On OS X, Apple wants you to make document based applications and gives you a fully working one with essentially no code. Honestly, I'm surprised that it took 916 lines of code to write the Mac UI - the UI is very simple with all the smarts hidden behind a text editor. There are a lot of "tricks" in that code to make the editing experience pleasant - most of which involves tricking the text editor into behaving itself.

This is the general lesson I've learned over the years: **writing iOS apps is a hell of a lot more work than writing Mac apps**. Enlightening? No. Honest? Yeah.

I wrote this app, as I do all my apps, using [C# and Xamarin](http://xamarin.com/ios) tools. I want to take a moment to thank them for their awesome product. Calca was a labor of love and it was wonderful to use my favorite tools to create it.
