---
layout: post
title:  "Two Hot New Features for Calca"
date:   2014-11-25 16:35:00 GMT
redirect_from:
  - /post/103559633748
  - /post/103559633748/two-hot-new-features-for-calca
thumbnail: "/images/tumblr/103559633748_0.png"
---


[Calca 1.3 has hit the Mac App Store](https://itunes.apple.com/us/app/calca/id635758264) and it contains two features that I hope you'll love: **plotting** and **quick entry**.

[![See Calca 1.3 on the Mac App Store]({{ "/images/tumblr/103559633748_0.png" | absolute_url }})](https://itunes.apple.com/us/app/calca/id635758264)

## Plotting


When Calca was first released, I was asked [a great question by Jason Brennan](http://nearthespeedoflight.com/article/2013_07_24_calca_review_and_interview):

> Do you think plotting is an essential function missing (most of) today’s “calculators”?


My answer was an unequivocal "YES!" followed by a lament for having cut it from v1.0 (#irony). A year passed and plotting became the [#1 voted feature for Calca](http://calca.uservoice.com).

Well plotting has finally arrived! You can now create as many plots as you want, each with multiple series. To do it, just pass a function to the plot function. For example, I can plot a sine wave at `5 Hz` with:

```csharp
plot(sin(5t * 2pi))
```


Passing multiple arguments generates multiple series:

```csharp
plot(sin(5t * 2pi), cos(5t * 2pi))
```


You can also plot data!

```csharp
plot([-1, pi, 0, 2])
```


The plots are displayed in the right margin of the application and support mouse tracing, zooming, panning, and even export as CSV or SVG.

All plots are displayed as 2D line graphs and are drawn using Calca's standard colors. The area under the curve - the *integral* - is always displayed as it gives quick visual indication of whether your data is positive or negative. Also, I think it looks pretty.

When you run Calca, *go to Help and choose Plots* for more details and examples.

More formatting options and more plot types will be added in the future, but I couldn't let this feature go unreleased any longer. I'm looking for your feedback! Now that we have plotting, I want to know what kind of plots you make so Calca can help you better in the future.


## Quick Entry


Calca is certainly a powerful "math document" editor, but sometimes you just want to do some quick math.

I often find myself in another app needing to do some quick calculations. Of course I could switch over to Calca, create a new document, do my work and switch back to that app. These aren't hard steps, but they wear on the soul.

The truth is, I found myself falling back to Spotlight for these quick computations - you can't argue with efficiency! But I missed Calca, I missed its symbolic nature, its units, its functions.

No More! Calca now has a UI mode that is very similar to Spotlight's. When Calca is running, simply hit the global keyboard shortcut **⇧⌘C** (of course you can change this) and you will be greeted with a little centered window all ready for you. Just start typing and Calca will happily perform your calculations and display the last one in the bottom half of the window. It makes computations in other apps fast and easy.

This has changed everything for me and is, honestly, my primary way of interacting with Calca now. I almost always start my work in the quick entry window and quite often just stay there. Only when the window gets to be too unwieldy do I hit **⌘N** to start working on a new document with that math. It's a great workflow that I hope you'll love.


## One More Thing


There has been this terribly annoying bug in Calca that resulted in text jumping around when editing long documents. That bug is dead now. Squashed. Eradicated. It will plague us no more.


### What about iOS and Windows?


They're coming!
