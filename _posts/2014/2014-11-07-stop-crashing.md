---
layout: post
title:  "Stop Crashing!"
date:   2014-11-07 16:28:00 GMT
redirect_from:
  - /post/102015518373
  - /post/102015518373/stop-crashing
---



**TL;DR** I wrote a script that lists functions in your Xamarin app that could cause it to crash. [StopCrashing.fsx on github](http://github.com/praeclarum/StopCrashing).

**We’ve all had apps crash.** It’s annoying, it’s stressful, and you feel cheated. Most people give up on apps that crash more than a couple times, or they leave 1 star reviews. Crashing is bad.

One of the benefits of using a managed runtime like .NET and Xamarin is that one of the most common causes of a crash - pointer corruption - is just not possible. Great! My apps should crash a lot less you think.

This is indeed true, but there is a catch: .NET uses exceptions as its error communication mechanism - just like every other modern runtime. That is fine, we all love exceptions.

Well, everyone loves them except old fuddy duddy operating systems from 1970 (i.e. every OS). They only know how to deal with unhandled exceptions by crashing. (Queue Hitchcock’s theme.)

What a world we live in! Our error communication mechanism actually causes our apps to shut down? Madness!

But that is indeed the world we live in. Hi, my name is Frank, and my apps crash. I hate it, I test and test but still manage to miss some exception handling in my code.

I’m tired of it. While crash reporting solutions exist, I would prefer to just not crash at all.

So I wrote a script.


## StopCrashing.fsx


I wrote a script that looks for possible ways the OS can call **into** my code. These entrypoints are the potential origins for crashes because there is no useful exception handler on the stack at that time.

What are these calls? Well, there are a lot of them:

1. Any time you override members of UI objects
2. Any time you export functions to the OS (as in, the responder chain)
3. Any callbacks you register with the OS
4. I’m sure there are more…

My script uses [Mono.Cecil](http://www.mono-project.com/docs/tools+libraries/libraries/Mono.Cecil/) to scan all the byte code of your application to find these entrypoints. (Well, I’ve only implemented #1 and #2. #3 is a harder problem to detect.)

It then detects whether you have any exception handling in that code. If you do not, it yells at you.

The idea is that we don’t want to bloat our software with exception handlers everywhere. This certainly makes coding more tedious but also introduces a lot of ambiguity. Instead, we still want to fail fast by leaving the majority of our code free of handlers. We will only catch exceptions where we must do so to prevent crashes.

[The script is available on github](http://github.com/praeclarum/StopCrashing). I put a little effort into making it easy to run: just pass it the path of your solution and it will go find the latest binaries automatically. It also returns an exit code so that it can be easily integrated into your CI server. (It is currently configured only to detect iOS and OS X calls. Android and Win RT support can be added if anyone is interested.)

Let’s run it on [Calca](http://calca.io) and see what it has to say:

`fsharpi --exec StopCrashing.fsx Calca.Mac.sln`

```csharp
30 UI METHODS IN Calca.Mac.sln NEED EXCEPTION HANDLERS
    Calca.Mac.Editor.InsertText
    Calca.Mac.Preferences.MakeKeyAndOrderFront
    Calca.Mac.TextEditorDelegate.DidChangeSelection
    Calca.Mac.TextEditorDelegate.GetCompletions
    Calca.Mac.TextEditorDelegate.LinkClicked
    Calca.Mac.TextEditorDelegate.ShouldSetSpellingState
    Calca.Mac.TextEditorDelegate.WillChangeSelection
    ...
```


Wow, it looks like there are 30 methods where an errant exception could crash my app! While I’ve done my best to make sure Calca doesn’t error, it’s just impossible to cover every possible situation. These 30 methods need to handle exceptions.

I have always *tried* to put good error handling in my code, but did a hap-hazard job of it. If the app doesn’t crash for my beta testers then it probably won’t crash for everyone else. Right? Well, no. I have come to the conclusion that you have to be more proactive. You have to defend against the unknown and this script ups my defenses significantly.


## Handling Exceptions


Well, if you’re going to handle errors, then you better do it responsibly. I find that a few types of functions repeatedly show up.

1. 

**Simple UI actions like clicking a button or entering text** These can be handled simply by catching all exceptions and popping up an alert to the user.
2. 

**System events** Since these can occur at random times, you may not want to alert the user (or at least not interrupt them with an alert). Random alerts popping up out of the blue will only confuse your users.
3. 

**Methods that return values** These are often feeding data into other APIs and need to be treated delicately. I try to allocate some safe defaults when the object is first constructed and then return those in error conditions.

Test your error handling! When I’m bored (that is, when I’m procrastinating over a difficult feature or bug) I often put random exceptions into my code just to test the error handling. What if this critical function fails, what will my app do? What if this seemingly innocent function fails, what will my app do? These little experiments can convince you that those 1 star reviews will at least be earned.

Of course, you can still benefit from crash loggers if your users don’t mind submitting error reports. The nice thing is, instead of crash reports, you’ll be delivering error reports.

As a last tip, may I remind you to be careful of loops. Displaying an error is bad and can annoy the user. Displaying the same error 100 times while your loop continually executes is catastrophically bad and you’ll be lucky if that user ever uses your app again.
