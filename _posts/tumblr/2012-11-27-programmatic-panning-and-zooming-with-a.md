---
layout: post
title:  "Programmatic Panning and Zooming with a UIScrollView"
date:   2012-11-27 17:57:00 GMT
redirect_from:
  - /post/36673962005
  - /post/36673962005/programmatic-panning-and-zooming-with-a
---



Let's say you have a zoomable and scrollable UIScrollView all setup in your app. Great! Good job. But how do you programmatically zoom in on something particular? (For example, you may want to pan and zoom into an object that was double tapped.)

UIScrollView exposes the **SetZoomScale** and **ScrollRectToVisible** methods, each of which can be animated. These are the methods that you will use but getting them to work together is a bit tricky. There was a lot of trial and error when I tried to get them to work together so I thought I would share my solution with you.

**Step 1. Determine a rect that bounds the content you want to zoom into.**

```csharp
var contentFrame = new RectangleF (x, y, width, height);
var scrollFrame = contentView.ConvertRectToView (contentFrame, scrollView);
```


First, we determine a bounding box for the content that we want to zoom into. Let's call it *contentFrame*.

We will also convert that bounding box to the coordinate system of the scroll view for use later.

**Step 2. Determine the new zoom scale.**

```csharp
var desiredZoomScale = 2;
var zooming = Math.Abs (scrollView.ZoomScale - desiredZoomScale) > 0.05f;
```


Now, determine the new desired zoom scale. This will depend on your app logic and perhaps on the contentFrame from Step 1. It's very app-specific, so I'll leave it at that for now.

We will also record whether the zoom needs to happen by comparing the desired zoom scale and the current zoom scale. We're going to need to know this to work our magic in Step 3.

**Step 3. Animate the pan and zoom.**

```csharp
if (zooming) {
    scrollView.SetZoomScale (desiredZoomScale, true);
}
scrollView.ScrollRectToVisible (scrollFrame, !zooming);
```


The trick here is to perform the zoom operation before the pan operation and to toggle the automatic animation system depending on whether we're zooming or not.

**If we are zooming:** Then tell SetZoomScale to animate and ScrollRectToVisible not to animate.

**If we are only panning:** Then don't call SetZoomScale but do tell ScrollRectToVisible to animate.

If you follow this simple procedure you will end up with nice smooth animations. If you don't, well, the results can be a bit jarring. Enjoy!
