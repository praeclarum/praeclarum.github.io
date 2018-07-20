---
layout: post
title:  "Calca - the text editor for engineers"
date:   2013-07-09 19:22:00 GMT
redirect_from:
  - /post/55019743936
  - /post/55019743936/calca-the-text-editor-for-engineers
---



[![image]({{ "/assets/tumblr/55019743936_0.png" | absolute_url }})](http://calca.io)

I am very pleased to announce the availability of [Calca](http://calca.io), my newest iOS app (and soon OS X app).

It's available on the App Store now for iPhone and iPad: it's cheap and crazy powerful. [Get it on the iOS App Store](https://itunes.apple.com/us/app/calca/id635757879?ls=1&mt=8).

Calca has been my obsession over the last three months - one that grew out of a frustration.

I work with mathematics every day. As a programmer, this is natural - it's very difficult to write an app without ever using addition! But I also write complicated apps - I often wonder if I will release an app that doesn't require the calculation of a [Jacobian](http://en.wikipedia.org/wiki/Jacobian_matrix_and_determinant). Other times, I'm just trying to avoid off-by-one indexing errors in my code!

These simple mathematics often get relegated to Sublime Text or Text Mate. I write out equations and then, line be line, manipulate them into useful forms.

This is tedious.

When it becomes too tedious, I resort to pen and paper. It's sad that I can write and manipulate mathematics on a sheet of paper more efficiently than I can with a computer in front of me.

One day, when it came time for me to write the Jacobian of a system of six functions each involving a quaternion with six different variables, I decided that neither Sublime Text nor the physical pen and paper were adequate. It was time to write a better tool.

And so, Calca was born with these goals:

1. As easy to use and as fast as a text editor.
2. Live updating like Excel.
3. Streamlined and forgiving math syntax.
4. Symbolic manipulation of expressions.
5. Plain text data.

Please allow me to elaborate on these goals:

**A text editor?**

Computer algebra systems are well defined: they're [REPLs](http://en.wikipedia.org/wiki/REPL). Like Perl, Python, and Prolog. You get one line of input and the program dumps text out on a console at you.

I have always had a love/hate relationship with REPLs. I love them for their convenience and speed. What's 2 + 2? That's about all I have to type to get an answer.

But one line for input? Even with [readline](http://web.mit.edu/gnu/doc/html/rlman_2.html), I get frustrated.

And then there is that issue of saving your work. Some REPLs make this easy, others laugh at you. It's telling that most serious users of REPLs couple them to a text editor so that their work can always be saved. Emacs seems designed for this. Newer editors, less so.

I figured that we might as well just build the REPL into the text editor itself.

In Calca, you can edit anywhere (it's a text editor!). You signal to Calca that you want it to do some work using the => operator. When Calca sees that, it figures out what you want it to do, does it, and prints the result to the right of the operator. Sounds weird huh? You need to try it (did I mention [Calca is cheap](https://itunes.apple.com/us/app/calca/id635757879?ls=1&mt=8)?)

Hanselman calls this the Anders operator. I have used it in all my work to mean "therefore". I'm fine with both names.

**Goal: Live Updates**

I also have an issue with REPLs when I make a mistake. Let's say I define a function, then do some calculations in Python:

>>> def average(a, b): return (a + b) / 3

>>> average(100, 200)

100

Oops, I made a mistake! Let's fix the average function:

>>> def average(a, b): return (a + b) / 2

>>> average(100, 200)

150

But I still have a lot of answers on the screen that reflect the old definition! This can be distracting or even harmful if my eyes wonder off.

This was fine in the 1960s when REPLs printed out to pieces of paper and companies could afford battalions of interns with whiteout. But it's the year 2013 and interns frustratingly demand to be paid - so I think it's time our computer tools deal with our errors better.

In that light, Calca works like Excel: as you make changes to earlier parts of the calculation, those updates are propagated throughout the entire document instantly. No interns needed. You really must [give it a try](https://itunes.apple.com/us/app/calca/id635757879?ls=1&mt=8).

**Goal: Humane Syntax**

Mathematical syntax is, well, politely speaking, without using cuss words, sometimes *slightly* ambiguous.

Humans love ambiguity and computers can't deal with it at all. Too bad for the computers, Calca uses a humane syntax.

The syntax was developed using an LALR(1) parser generator.  LALR(1) parsers can only look ahead one symbol at a time. This necessitates creating unambiguous and simple grammars. That's my fancy way of saying that Calca's syntax is not complicated and is very easy to learn.

In addition to this, Calca has lots of convenience syntax. For example:

* **f = 2x** is a function even though I didn't bother with any parenthesis
* **2x** is shorthand for **2 * x**
* You can type **33%** instead of **0.33**
* Numbers can be written with grouping separators because I'm getting old: **100,033,234.56**
* Names can include spaces so we can write **dist to the moon** instead of **dist_to_the_moon** or **distToTheMoon**

Take a look at the [Examples](http://calca.io/examples/) and [Reference](http://calca.io/support/) to get a feel for its syntax.

**Goal: Symbolic Manipulation**

Sometimes, we just don't know something. What is the tax rate? How far is the moon? How many clowns will fit in that car?

We use variables for these quantities, but most calculation systems (desktop calculators, programming languages, etc.) require we know a value for all variables. Ludicrous!

Calca loves undefined variables. It will treat them, properly, as unknowns and otherwise just carry on.

For example let's say we're trying to understand how much money the man takes from us. I know how much I, ostensibly, make per year, and I know how much my check is per month. We can write this equation:

**(yearly salary / 12) * tax percent / 100 = monthly take**

There is only two numbers in that equation and three variables, but we can still ask Calca for the tax rate:

**tax percent => 1200monthly take/yearly salary**

Of course, if we give those variables numeric values, then Calca can reciprocate with numeric answers.

**Goal: Plain Text**

All of Calc's documents are stored as plain UTF-8 text. This makes them as easy to move around and manage as possible. Git loves plain text. Email loves it. They can be edited in Emacs or vi. All your Text Expander shortcuts work.

There is nothing more frustrating than performing a lot of work only to realize you can't share it. Proprietary file formats are plagues imposed on us by evil software developers. With plain text, you control the data, not Calca.

**Conclusion**

Thanks for reading!

Calca is the tool that I needed for years and I am so happy that I now have it to assist me day to day.

I hope that I convinced you that it will be useful for you too. So [go grab a copy](http://calca.io) and let me know!
