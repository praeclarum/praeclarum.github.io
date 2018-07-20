---
layout: post
title:  "Fashionable REPL Prompts"
date:   2017-02-27 17:09:52 GMT
redirect_from:
  - /post/157784591258
  - /post/157784591258/fashionable-repl-prompts
---



I was writing a new language the other day and I thought, “this puppy needs a REPL”!

But before I could write one, I had to decide how it would look and behave. I mean, I knew the basics: take something in, execute it, then display the result. But how do you open the help? How do you handle multi-line input? Can I use terminal colors? **What does the prompt look like?**

To answer that last one, I took a quick survey of my favorite languages - turns out they've all coalesced to the `>` prompt, but there are some fun variations:


## F#


```csharp
> 2 + 3;;
val it : int = 5
```


F# is my favorite language, but the REPL is a bit busy for me. First the language dictates this weird crying emoji (`;;`) in input and the result is always encumbered by `val it :` noise. But I still <3 you F#.


## C#


```csharp
csharp> 2 + 3
5
```


Much cleaner! No semicolons, and just the answer. Well C# does show its vanity a bit with its name announcement on each line - but heh, it deserves it.


## Python


```csharp
>>> 2 + 3
5
```


Elegant and bold at the same time. The `>>>` means Python, but you don't actually say "Python". So hipster, so cool. I'm sure I could copy `>>>` as remixing is hip these days; but no, I'd be trying too hard.


## Ruby


```csharp
irb(main):009:0> 2 + 3
=> 5
```


OK, I get what you're going for here Ruby. Part of me even likes it. But no. Too much. I would expect this kind of complexity and technical jargon when logging into my refrigerator - but my dev environments should have a little more refinement.

On the plus side - Ruby outputs the answer in yellow. I'm totally stealing that.

Oh and geeze, they use the same symbol as [Calca](http://calca.io) - so how could I not love that?


## JavaScript


```csharp
> 2 + 3
5
```


Look at you. Simple, reasonable, well thought out. It's like staring at an oil on canvas painting containing only the lowercase Helvetica `a`.


## Conclusion


While we have standardized on `>` as the one prompt to rule them all, there is a fair amount of diversity as to what comes before it.

I'm a fan of simplicity and in the end, I went with C#'s `vanity>` prompt. Cause, like, I'm vain.
