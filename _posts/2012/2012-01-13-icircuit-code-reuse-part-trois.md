---
layout: post
title:  "iCircuit Code Reuse Part Trois"
date:   2012-01-13 21:26:04 GMT
redirect_from:
  - /post/15789866032
  - /post/15789866032/icircuit-code-reuse-part-trois
---



I have recently finished porting [iCircuit](http://icircuitapp.com) to Microsoft Windows Phone 7 (WP7) and wanted to see how I performed on the code reuse front.

I use [MonoTouch](http://xamarin.com/monotouch) to do iOS development, and that was the first version of the application. Some months ago, I also released a Mac version of the app using [MonoMac](http://www.mono-project.com/MonoMac). It was then that I [gave a presentation](http://www.infoq.com/presentations/3-Mobile-App-Development-Problems) giving some general stats on the amount of code I was able to reuse between the two versions.

Now, with a third version, it's time to repeat the report. This is also the first version of the app not written using Mono tooling. That's right, I used a PC. It was weird.

I wrote a little script to calculate the stats I'm about to present. Here is [gist of that script](https://gist.github.com/1608597) so that you can decide for yourself if these stats are worthwhile. All the compiled code for each project got put into two buckets: unique code and shared code. Unique code is code used only for that project while shared code is code that was used in more than 1 project (not requiring that it be used in all projects). The majority of the shared code comes from the iCircuit cross platform library, but also includes things like shared graphics drivers and file system abstractions.

The raw output from that script is as follows:

``

```csharp
app     t       u       s       u%      s%
Mac     29954   3901    26053   13.02 % 86.98 %
WP7     30744   5760    24984   18.74 % 81.26 %
iOS     37518   11532   25986   30.74 % 69.26 %
```


``

(t is Total lines of code, u is Unique lines of code, and s in Shared lines of code. Lines of code include comments and white space. I have a fairly consistent coding style, so this simple metric is adequate.)

And here are some pie charts to make that data easier to visualize.

![image]({{ "/assets/tumblr/15789866032_0.png" | absolute_url }})

Green code is shared, and Red code is unique to the app. Lines of code and percentages are shown.

The iOS version of the app has the most amount of platform specific code, 11,532 lines worth. I can give you 1,000 reasons for this, but the gist of it is that the platform has the most effort put into it and includes an iPhone version along with an iPad version.

The Mac version did much better with only 3,901 lines of unique code needing to be written. It is particularly small because it not only shared the cross-platform code, but was also able to share a lot of driver code with the iOS version: things like the audio drivers and graphics drivers were directly shared between the two platforms thanks to the awesome work of the Mono team and Apple.

I was hoping the WP7 version would be even less, but, it turns out, it still took a good amount of effort. 5,760 lines of code had to be written accounting for nearly 20% of the code base. New graphics drivers, and new audio drivers had to be written to support the Silverlight programming model along with a lot of IsolatedStorage tomfoolery. Now, when I release Windows 8, WPF, and Silverlight versions of the app, a lot of that code will also get reused, but for now it stands on its own.

Overall, I'm quite pleased with these results. I always talk about the 80/20 rule being applied to the cross-platform/platform-specific code split and so I'm very happy that the number back me up pretty well.

I can't wait to get on to the next platform!
