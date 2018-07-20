---
layout: post
title:  "A Two-year-old Bug Found by Microsoft"
date:   2012-01-20 23:43:00 GMT
redirect_from:
  - /post/16194355039
  - /post/16194355039/a-two-year-old-bug-found-by-microsoft
---



I submitted the Windows Phone 7 version of iCircuit to Microsoft a few days ago and was shocked (shocked!) that it failed certification by their testers.

I was in denial, how could this be? The engine is two years old, and, while it has issues, is very stable on a variety of platforms. Add to this the fact that I couldn't reproduce the bug right away, I was stymied.

But after some patience and Diet Coke, I discovered that not only did they discover a bug in the engine, but one that's been there since its initial release. That means it has been hidden from thousands of users and many many rounds of testing. I have to hand it to Microsoft for finding it.

I would like to discuss the nature of the bug in some detail here since I have never run into something of its kind before.

As with all numerical apps, the bug lied in floating point precision problems.

The Scope in iCircuit automatically determines a range for the value axis of the plot. If the signal ranges from -5 to 5 V, then the plot shows that range. To do the actual plotting it calculates a pixel ratio:

**PPV = Height of the Plot / (Max Value - Min Value)**

But what happens if you have a DC signal that ranges from, say, 5 V to 5 V? You would get a divide by 0 in the calculation of PPV. Being an electronics simulator where DC signals occur all the time, iCircuit had to have code to handle this case. And so, pseudo min and max values were introduced:

**DV = Max Value - Min Value**

**if (DV == 0) { DV = 1e-6; }**

**Pseudo Min Value = Min Value - DV**

**Pseudo Max Value = Max Value + DV**

PPV is then calculated as above but using these pseudo variables instead of the real values. DV is set to a value that makes the plot look decent but is any arbitrary value greater than 0. You can think of DV as creating a little gap around the constant values.

This is the code that's been in iCircuit since v1.0 and has worked quite well. Until Microsoft got their hands on my app.

Their tester decided to hook up an LED directly to a DC source. All electricians know that this is a bad idea - you'll burn out the LED since diodes have very little internal resistance. You always put some resistance in series with the LED. But he didn't.

LEDs do have some internal resistance that iCircuit was simulating to produce a current of over **1.9e19**. That's a huge current! Impossibly huge, but theoretically correct.

When the tester went to observe that current on the scope, it was determined to be a DC value (DV == 0) and the pseudo values were calculated. Kinda.

It turns out that **1.9e19 +/- 1e-6 == 1.9e19** thanks to floating-point imprecision.

Oh my. That means that the pseudo values were still the same and we hit the divide by 0 bug. When I tried to plot a NaN value (some more math is done on PPV), an unhandled exception was triggered. Oh my indeed.

That is the crash that the Microsoft tester found. Good job, man.

Now the solution. Instead of using a fixed minimum DV value of 1e-6, I calculate it as

**Min DV = |Max Value| * 1e-6**

and DV becomes

**DV = max(Min DV, Max Value - Min Value)**

This makes sure that DV will be substantial enough to ensure Pseudo Min Value and Pseudo Max Value are different and the divide by 0 is avoided.

Well, almost, there's still a bug there, but I'll let you discover it.
