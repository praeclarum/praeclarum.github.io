---
layout: post
title: "Oops, I Wrote a C++ Compiler"
---

**TLDR;** I wrote a .NET library that can compile C/C++ code into
a byte code that it can also interpret. It is used in my
app [iCircuit](https://icircuitapp.com) to simulate Arduinos.


## Arduino Support for iCircuit

The most requested feature for iCircuit, for years, has
been to support Arduino components.
I agreed with all my users that it would be an amazing
feature, but it was also a pretty big request.

I could easily add a component that looked like an Arduino -
had all the right pins, maybe even blinked an LED. However,
what people really wanted was a *programmable* Arduino.

Arduino's are programmed in C++ with a small base library known
as "Wiring". Therefore, in order to simulate an Arduino, I'm
going to need a C/C++ compiler and I'm going to have to re-implement Wiring. Oh, and that compiler needs to
run on iOS, integrate into my circuit simulation, handle
bad code such as infinite loops, and all
work within the sandbox (which means interpretation instead
of real execution). Like I said, it was a big request.

As tough as all that sounds, I still personally wanted the
feature. Way back in 2010, I started work.

First I looked around for small C++ compilers and
interpreters that I could
get to work on iOS. The research was grim as no compiler
met all my requirements. Some would be nice and small but
only emit X86 code meaning I would have to write an X86 simulator. Others were so big and had so many dependencies
that I just gave up trying to get it to compile for iOS.

Fortunately in 2010, I was full of hubris and I figured now was the time in my life to write a C++ compiler. Oh how foolish...


## The Halcyon Days of 2010

Compilers and Interpreters are, in principle, very simple
programs. I'd written a bunch of interpreters at this point
in my career and even wrote a couple basic compilers for very
small languages. I knew that C was a complex language, at
least syntactically (with all its type declarations) but that
its semantics - its model of computation - was rather basic.
I considered myself a good C++ programmer and thought I knew
the language inside and out. How hard could it be?

Like I said, hubris.

And so I started writing my compiler in C#.
I started by writing a C compiler because C++ is a monster of a
language. Most Arduino programs only use C features (I thought),
so it seemed like a reasonable starting point.

C is nice because you can actually parse it using
a *grammar definition* - a file that states the syntax of
the language. Grammars are great because you can use a code
generator to create a parser for your language.

Back in 1985 someone posted a [YACC grammar definition for C](https://www.lysator.liu.se/c/ANSI-C-grammar-y.html)
which served as my starting point.
I then used the [jay parser generator](https://www.cs.rit.edu/~ats/projects/lp/doc/jay/package-summary.html) (the same tools used to generate the mono C# parser)
as my parser generator.

The process of creating the parser went very smoothly thanks
to this. The real work involves creating C# syntax classes
that mirror the grammar. These classes would be the Abstract
Syntax Tree (AST) of my compiler. You can see the results
of this work by [looking at my modified grammar](https://github.com/praeclarum/CLanguage/blob/master/CLanguage/Parser/CParser.jay).

And this is the point where I realized C was a bit more complex than I liked to think.

My syntax classes were filled with scary names like "abstract
specifiers", "type specifier", "type qualifier", "multi declarations", and on.

I was scared, but combatted it by writing a bunch of unit tests.
I figured, yes this problem is hard, but it's just big -
maximum effort would be needed but there was an end in sight.

And so I pressed on. The next step to writing a compiler is
discovering definitions in code. Variable definitions, function
definitions, type definitions, all that. C and C++ are a little
wild because you can declare things multiple times but you can
only define them once.

Once I found declarations and definitions, I needed to build
a type system. Thankfully, at first glance, C's types are very
basic. You have the machine types (ints and floats), 
structures, and, uh oh, pointers. The compiler would have to munge
around all the code and assemble and unify all these types.
I spent days and days just getting the integers right.

Now it's time for the compiler to earn its keep and emit
executable code. Most C compilers would emit X86 or ARM assembly.
However, there was no point in doing that as I can't natively
execute code on iOS. I decided to instead devise my own
byte code that would be easy to interpret.
I figured that if I controlled the compiler and the byte code
then I could arrange things to make the interpreter simple
and yet still expressive.

Most of those early decisions paid off well and after a few weeks
of work I was able to compile and interpret the most basic of
Arduino programs.

But that victory was met by a cold realization
of just how much more work was left to be done.

My compiler was working but it could only do math with 32 bit integers. My clever interpreter was stack based - which,
turns out, is not at all compatible with C's flat memory model.
I had no idea how I would make real pointers work. I still
didn't have support for structs. And this was all just for the
C compiler - I still needed to add C++ features!

And so, sadly, I gave up. It's never easy to admit, but sometimes
problems are just too big for you.


## 2018

Eight years is a long time. 




