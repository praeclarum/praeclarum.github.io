---
layout: post
title:  "Await in the Land of iOS - Drag-n-drop"
date:   2013-03-13 00:54:00 GMT
redirect_from:
  - /post/45231096776
  - /post/45231096776/await-in-the-land-of-ios-drag-n-drop
---



Today is a big day, [Xamarin has released](http://blog.xamarin.com/brave-new-async-mobile-world/) (in alpha) support for the [await keyword](http://msdn.microsoft.com/en-us/library/vstudio/hh156528.aspx) in C#.

To celebrate, I thought I would write a few articles describing the ways I plan (and now am) taking advantage of that support.


### Async code without await is code written inside out


Asynchronous programming has taken over our lives. We allowed it to because it benefits software ranging from high performance servers all the way down to responsive UIs on phones. That is to say, it’s good for machines and users.

But async programming comes at a cost for us programmers. It often asks us to mangle our beautiful procedural code into state machines or continuation passing style. We can’t just say “Do A, then B, then C”:

```csharp
DoA ();
DoB ();
DoC ();
```


Instead, we have to write the code using callbacks:

```csharp
DoA (errA =>
    if (errA != null) HandleError (errA)
    else DoB (errB =>
        if (errB != null) HandleError (errB)
        else DoC ()));
```


That code is pretty horrendous compared to the procedural code. It’s too long. It turns our error handling from exception-based to return code-based. The error handling also hides the control flow. It is, quite simply, a mess.

The C# await keyword allows us to write asynchronous procedural code in our preferred style so long we don’t mind a few keywords:

```csharp
await DoA ();
await DoB ();
await DoC ();
```


What a breath of fresh air!


### What else is inside out?


Await has paid for itself by greatly simplifying our async code. But it made me wonder, what other bits of code am I writing inside out? In other words, what other APIs do I often interact with that are async - that happen in user time, not CPU time?

Event handlers for touch events sprung immediately to mind.

I wondered about an old friend of mine, the drag-n-drop procedure. Dragging objects on the screen is simple:

1. When a touch begins, look to see if it touches any subviews
2. If it does, then move that view whenever the touch moves until the touch ends

Below is code that I have written 1,268 times (I checked) to implement that procedure in iOS:

```csharp
class DragInfo
{
    public UITouch Touch;
    public UIView View;
}

readonly Dictionary<UITouch, DragInfo> drags =
    new Dictionary<UITouch, DragInfo> ();

public override void TouchesBegan (NSSet touches, UIEvent evt)
{
    foreach (var touch in touches.ToArray<UITouch> ()) {
        var loc = touch.LocationInView (this);
        var view = Subviews.FirstOrDefault (x => x.Frame.Contains (loc));

        if (view != null)
            drags[touch] = new DragInfo {
                Touch = touch,
                View = view,
            };
    }
}

public override void TouchesMoved (NSSet touches, UIEvent evt)
{
    foreach (var touch in touches.ToArray<UITouch> ()) {
        DragInfo drag;
        if (!drags.TryGetValue (touch, out drag))
            continue;

        var loc = touch.LocationInView (this);
        var ploc = touch.PreviousLocationInView (this);

        var fr = drag.View.Frame;
        fr.X += (loc.X - ploc.X);
        fr.Y += (loc.Y - ploc.Y);
        drag.View.Frame = fr;
    }
}

public override void TouchesEnded (NSSet touches, UIEvent evt)
{
    foreach (var touch in touches.ToArray<UITouch> ()) {
        DragInfo drag;
        if (drags.TryGetValue (touch, out drag))
            drags.Remove (touch);
    }
}
```


It implements multi-touch dragging of views by using some fields to store state between touch events.

How well does this code align with my dragging procedure above? I wrote the code as well as I ever have (i.e. my style hasn’t changed since programming Windows 3.1). But it still has very little resemblance.

The events become the main focus of the code instead of the procedure. I had to introduce a data structure and some mutable, essentially global, data that 3 different functions work with in a coordinated way. Since async methods can be called any time, I just have to hope that I worked out all the possible interleaves in my head (I probably didn’t).

Can we do better? I rewrote that code using `await` to find out.

```csharp
// Step 1: Watch for dragging starts
async void WatchForDrags ()
{
    for (;;) {
        var began = await this.GetEventAsync<TouchEventArgs> ("TouchBegan");

        foreach (var t in began.Touches) {
            var loc = t.LocationInView (this);
            var view = Subviews.FirstOrDefault (x => x.Frame.Contains (loc));

            if (view != null)
                DragView (t, view);
        }
    }
}

// Step 2: Carry-through with the drag until it's over
async void DragView (UITouch touch, UIView view)
{
    for (;;) {
        var moved = this.GetEventAsync<TouchEventArgs> ("TouchMoved");
        var ended = this.GetEventAsync<TouchEventArgs> ("TouchEnded");

        var ev = await Task.WhenAny (ended, moved);

        if (!ev.Result.Touches.Contains (touch))
            continue;

        if (ev == ended)
            return;

        var loc = touch.LocationInView (this);
        var ploc = touch.PreviousLocationInView (this);

        var fr = view.Frame;
        fr.X += (loc.X - ploc.X);
        fr.Y += (loc.Y - ploc.Y);
        view.Frame = fr;
    }
}
```


Using a little trick function, I was able to turn normal .NET events into tasks. The idea is simply to subscribe to the event long enough for it to fire once and only once. That one little trick allowed me to increase the readability of the code.

What was 3 functions, 1 data structure, and 1 chunk of memory has been turned into two procedures that align very well to the English procedure above.

I love that this code doesn’t leak its state out into the class containing it. `DragInfo` is a failure of encapsulation - the data required by the events Began, Moved, Ended was available to all other methods in the class. Now, there is no leaked state. Instead, the state is held in a clojure created when DragView is called by WatchForDrag.

I love that the new code is procedural - lines higher up on the screen happen earlier in time. I love that it processes events in the order that it cares about - no other order.

This is straight procedural code that executes asynchronously in response to user interactions. The machine is happy because I’m not spawning threads, the user is happy because the UI is responsive, and I am happy because I can write code the way I think, not the way I’m required to just to satisfy those other two conditions.

I hope you found this a little interesting. I have a lot more ideas on how to take advantage of `await` so I hope you’ll stay tuned.


### A little trick function


The GetEventAsync is the magic that allows me to await events. Here is a rough implementation of it.

```csharp
public static Task<T> GetEventAsync<T> (this object eventSource, string eventName)
    where T : EventArgs
{
    var tcs = new TaskCompletionSource<T>();

    var type = eventSource.GetType ();
    var ev = type.GetEvent (eventName);

    EventHandler handler;

    handler = delegate (object sender, EventArgs e) {
        ev.RemoveEventHandler (eventSource, handler);
        tcs.SetResult ((T)e);
    };  

    ev.AddEventHandler (eventSource, handler);
    return tcs.Task;
}
```

