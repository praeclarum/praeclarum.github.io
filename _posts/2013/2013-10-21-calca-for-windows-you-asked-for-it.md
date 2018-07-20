---
layout: post
title:  "Calca for Windows, you asked for it!"
date:   2013-10-21 22:25:00 GMT
redirect_from:
  - /post/64717121779
  - /post/64717121779/calca-for-windows-you-asked-for-it
---



![image]({{ "/assets/tumblr/64717121779_0.png" | absolute_url }})

[Calca for Windows](http://calca.io/store/calca-for-windows) has been the [#1 asked for "feature"](https://calca.uservoice.com/forums/216746-general/suggestions/4247892-windows-version) since [Calca for iOS](http://calca.io) debuted in July. Today, I'm happy to announce that it's finally available for purchase! For just $9.99 you can own this powerhouse of a calculator.

**A very Windows app**

Calca for Windows was designed to be fast and light weight. It ships as a single executable that doesn't need to be installed (don't worry, there's an installer too if you want it) and presents a minimalist user interface. There is no drama of loading a big IDE or massive spreadsheets. Calca pops right up and begs you for something to do.

And, oh, it works on Windows XP. Yep, that's right: Calca was designed to work where you have to work, no judgements.

If you're just getting started with Calca, make sure you read the Introduction and Examples under the Help menu to get started.

**A more powerful engine**

One of the most exciting things about this release is that it ships with version 1.2 of the Calca engine. In addition to numerous bug fixes, this version also has fantastic units and currency support.

You can easily convert simple values:

> 12 tbsp in cups => 0.75 cups


Units have been implemented in a very general manner allowing you to do some pretty advanced computations.

Here is one of my favorites: how to calculate how much water falls on your land throughout the year.

> land area = 0.25 acreavg annual precip = 36.15 inch / yeardaily rain accumulation     = avg annual precip * land area in gallon/day     =>671.9012 gallon/day


In order to perform this calculation, Calca had to know how to convert between acres, inches, days, years, and gallons and it does it all with a minimal amount of typing.

There are lots of other improvements, but since this is a new release, I won't bore you all with details here. But watch this blog as I will be pointing out nice new features from time to time.

(And don't worry Mac and iOS friends, updates are being submitted to Apple this week so that you will get these new features too.)

**Code reuse numbers**

For my developer friends out there who like to keep tabs on me, I was able to **reuse 18,000 lines of code** from the previous Calca versions and only had to **write 3,000**. The majority of that code is implementing a document-based application architecture.

Here the code reuse stats in picture form:

![image]({{ "/assets/tumblr/64717121779_1.png" | absolute_url }})

**86% code reuse** is great, though I do with it was in the 90s. The Mac is able to achieve this because of the powerful Cocoa library that has first-class support for document based applications. On Windows, I used Win Forms and had to implement all the file management by hand just like with iOS. Speaking of iOS, it's numbers have grown a bit because I have some nice surprises in store for its updates.
