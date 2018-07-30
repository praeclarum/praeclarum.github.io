---
layout: post
title:  "Await in the Land of iOS - Scripting Users"
date:   2013-03-13 17:47:55 GMT
redirect_from:
  - /post/45277337108
  - /post/45277337108/await-in-the-land-of-ios-scripting-users
thumbnail: "https://img.youtube.com/vi/TmAOeGToFeE/default.jpg"
---

**TLDR;** I show how to use the `await` keyword in C# to build an interactive help system.


### Help!


We all know our UIs are intuitive and that our icons are perfectly chosen, but why leave learning the UI to chance?

The traditional solution to this problem is to provide help files. These are great, except no one reads them (at first!). If they get frustrated with your UI, they instead press the Home button and look at Twitter.

An alternative help system, one that I think is superior, integrates with the UI of the application. Think “tooltips on steroids”.

Check out the help system in Apple’s iPhoto app:

![image]({{ "/images/tumblr/45277337108_0.png" | absolute_url }})

We have attractive tooltips describing the actions of the various toolbar items.

This is nice, but what about accomplishing *tasks*? Tasks often take multiple steps to accomplish use different parts of the UI. We write out tasks in our help files:

> To search for pictures of cats:Enter the term “cats” in the search box.Tap the “Search” button.


This has the failing that the user has to swap between the app and the help file.

I want a help system that walks a user through a specific task. And I want it to do it in a way that doesn’t force the user to search for “cats” (100% automated) but let’s them accomplish the task with their own input and decisions. Such a system could also be used to give introductory tutorials - the “getting to know the app” task.

Ambitious? yes.


### Await


But I don’t know how to write it (frown face). Well, I don’t know how to write it without making a mess out of all my UI code that is already, well, messy. Instrumenting messy code with help file info sounds more than scary.

It be nice (understatement) if I could just take those help file directions and turn them into code.

Await has helped me see a nice way to implement it.

```csharp
async Task ShowTheUserHowToSearch ()
{
    await Tutorial.EnterText (searchField, minLength: 3);
    await Tutorial.Tap (searchButton);
    await Tutorial.Congratulate ("Now you know how to search.");
}
```


This is an async function that uses a library I call [Tutorial](#tutorial) to execute help file instructions. I tend to call these things “scripts”.

The method first asks the user to enter some text, then to tap a button. If they take those two tough steps, then you congratulate them!

The system automatically highlights the control with some text while waiting for them to take a step to egg them on.

This is my programmer art aspiring to be Apple’s beautiful UI in iPhoto:



<iframe frameborder="0" height="315" src="http://www.youtube.com/embed/TmAOeGToFeE?rel=0" width="420"></iframe>



### Tutorial


Thanks to the await keyword and async methods, it was shockingly easy to implement the Tutorial class. It comes out at a measly 40 lines of code if you ignore my ugly programmer art (100 lines otherwise).

Let’s look at directing users to tap a button:

```csharp
public static class Tutorial
{
    public static async Task Tap (UIButton view)
    {
        Highlight (view, "Tap");
        await view.GetEventAsync ("TouchUpInside");
        Unhighlight (view);
    }
}
```


The first line highlights the button. We then wait for a `TouchUpInside` event to fire. Once it has, we simply remove the highlight from the button. (`GetEventAsync` is from the [previous post](http://praeclarum.org/post/45231096776/await-in-the-land-of-ios-drag-n-drop).)

This code is quite straight-forward. Surprisingly straight-forward given the amount of work that it’s doing for us. It’s keeping track of state, UI elements, waiting for events. All in 3 lines of simple code.

Let’s cover the rest before I get too choked up…

```csharp
public static async Task EnterText (UITextField view, int minLength = 1)
{
    Func<bool> ok = () =>
        view.Text != null && view.Text.Length >= minLength;
    if (ok ())
        return;

    Highlight (view, "Enter Text");
    view.BecomeFirstResponder ();

    for (;;) {
        await view.GetEventAsync ("AllEditingEvents");
        if (ok ()) {
            Unhighlight (view);
            return;
        }
    }
}
```


Entering text is just as easy but contains a bit more logic. We want to watch the UITextField every time the text changes. To accomplish this, we just keep checking the `Text` property whenever an editing event has occurred.

We also skip this step if there is already valid text. Whether this is a good idea or whether the library should force them to edit the text anyway is up for debate.

If you read my [earlier blog post on await](http://praeclarum.org/post/45231096776/await-in-the-land-of-ios-drag-n-drop) then you will notice this pattern emerging of infinite loops used to wait for potentially long periods of time. Please keep in mind that this isn’t polling - it just *looks like polling*. We are still event based and there will be 0 CPU usage due to this infinite loop. Ahh, magic. As this is only my second day of experimentation, I don’t know if this is a good pattern or not. But for now, it works well and is pretty readable.

Congratulating the user is also simple:

```csharp
public static async Task Congratulate (string message)
{
    var a = new UIAlertView ("Congratulations!", message, null, "OK");
    var clicked = a.GetEventAsync<UIButtonEventArgs> ("Clicked");
    a.Show ();
    await clicked;
}
```


We simply display an alert with a nice message and wait for the user to dismiss it.

We can build upon `Congratulate` to ask the user a question:

```csharp
public static async Task<int> Ask (string question, params string[] responses)
{
    var a = new UIAlertView (
        "", question, null,
        responses.First (), responses.Skip (1).ToArray ());
    var clicked = a.GetEventAsync<UIButtonEventArgs> ("Clicked");
    a.Show ();
    return (await clicked).ButtonIndex;
}
```


Using `Ask`, we can have a nice dialog with the user:

```csharp
async Task ShowTheUserHowToSearchWhileBeingAnnoying ()
{
    var r = await Tutorial.Ask (
        "Do you want to search for cats?",
        "No", "Yes, cats!");
    if (r == 1)
        searchField.Text = "cats";

    for (;;) {
        await ShowTheUserHowToSearch ();

        r = await Tutorial.Ask (
            "Do you want to play again?",
            "No", "Search!");
        if (r == 0)
            break;

        searchField.Text = "";
    }
}
```


This script demonstrates that we can:

* Compose help tasks together easily
* Use all of C#’s control flow structures while disregarding time
* Interact with the user contextually without managing state variables

It’s a lot like scripting an application UI for testing. Except, now, we’re scripting the user and reacting to them.

And it’s all possible thanks to await. Try writing that code using event handlers. I’ll wait…



<iframe frameborder="0" height="315" src="http://www.youtube.com/embed/yTD-pKe9vRw?rel=0" width="420"></iframe>



### Thanks


My thanks go out to everyone who worked on async both at Microsoft and on the Mono project for building such a useful and versatile language feature.

I want to re-iterate that I don’t know how, or couldn’t implement this kind of help system without async. Managing the state of many possible interactions (imagine two of these help tasks running simultaneously) without async hurts my head. Not only does async provide a solution to the state management problem, but it does so in a way that is simple and doesn’t touch your UI code.
