---
layout: post
title:  "Coding in Your Head"
date:   2015-05-28 14:52:24 GMT
redirect_from:
  - /post/120107368103
  - /post/120107368103/coding-in-your-head
tags: article
---



I’m terrible at coding interviews - some busy bee dusts off a tricky algorithm that they studied in college and asks you to (1) originate it from a poorly stated problem and (2) live code it in front of them.

This isn’t how I work. Like most programmers who survive more than a few years in this business faced with a novel or difficult problem, I do the majority of my design work in my head - slowly.


## Realm of Endless Possibilities


The problem gets repeated endlessly: “The user wants to accomplish X, Y, and Z - I will need to talk to data sources I, J, K - I will use algorithms A, B, C - they are connected in this configuration or that - information will be on a screen that looks like...”

I try out all the permutations of data structures, objects, their relationships to one another, algorithms that I already know, and algorithms that I note to seek out. I think through the user interface - attempting to limit the number of choices the user has to make to do repetitive tasks while still trying to giving them a new power.

Steeped in years of OOP programming, all this design work culminates in an object schema in my head. Known classes and their relationships to other classes are built and toyed with. I refine this graph by running many algorithms across it to see how nasty my layers of abstraction and encapsulation make moving data around (remember, in the end, the most important thing to your program is the data - not how you represent it). I look at it to see how easy it will be to extend or flat out replace in the future.

This is a slow process. It’s why I have a list of 100 “potential next apps”. They’re up in my head (or at least a few top candidates) while I toss them around and poke and prod at their code.


## Coding It


Once a design is deemed robust, useful, and interesting enough, it’s time to sit down and code it. At this point you are basically limited by your programming language. This is why I’m a programming language nerd and relentless critic.

I don’t care about powerful programming languages because they save me from typing. I care about them because they **allow me to get closer to my mental design** than less powerful languages.

Designs of the mind are necessarily abstract - unconcerned with particulars of language. My “head design language” is just objects, interfaces, methods, properties, and events. Call this OOP 1.0. (As I learn functional programming, my language is slowly turning to records, abstract data types, interfaces, and functions.)

When I sit down to write these, any boilerplate that the language forces on me becomes an annoyance. C++ and Objective-C that require designing a memory strategy are profoundly annoying (I can barely get my own designs right, and now the fracking computer needs help too?). C#’s lack of metaprogramming and first class events is another annoyance. F#’s single-pass compiler that makes you order every declaration and even your source files (seriously, what decade is this?) is, you guessed it, annoying. Even trivial syntax gets annoying at this point - why do I have to write all those silly characters? { ; } oh my.

The tools we use also become obstacles. Intelligent IDEs that are intended to make coding easier become enemies with every spinning beach ball - with every hidden setting - with every error message. Imagine trying to create an intricate sand castle on the beach during a hurricane. No wonder text editors such as Sublime are such hits.

So your beautiful mental design gets compromised into some language or another. This is why we call it coding - **we are encoding a design into some barbaric text format** that only highly paid professionals and intelligent 13 year olds can understand. Anyway...

That’s all to say that it’s best to burn through all the bad designs in your head so that only the decent ones have to suffer this transition to code.


## Some More Thoughts


**It’s a slow process** but it can’t be sped up. No, test driven development is not an answer. TDD causes you to hash out a design - but one that’s biased to one consumer - the tests. It neglects the most important consumer - the end user. Also I am happy to throw out a design that I’ve been mulling over for a week. I have never once seen a TDD advocate throw away a week’s worth of Asserts - no they just get painfully “refactored” into the next design option.

**It’s not a perfect process** because your initial designs are never right. Certainly it saves you from writing endless numbers of throw away prototypes before you settle on a good design - but it won’t be a perfect design. It will have to be changed once you’ve implemented the app and learned what the app really is and how people really use it.
