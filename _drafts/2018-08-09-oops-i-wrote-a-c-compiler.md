---
layout: post
title: "Oops, I Wrote a C++ Compiler"
---

**TLDR;** I wrote a .NET library that can compile C/C++ code into
a byte code that it can also interpret. It is used in my
app [iCircuit](http://icircuitapp.com) to simulate Arduinos.


## Arduino Support for iCircuit

The most requested feature for iCircuit, for years, has
been to support Arduino components.
I agreed with all my users that it would be an amazing
feature, but it was also a pretty big request that I wasn't
sure I could complete.

I could easily add a component that looked like an Arduino -
had all the right pins, maybe even blinked an LED. However,
what people really wanted was a *programmable* Arduino
integrated into the circuit simulator.

Arduinos are programmed in C++ with a small base library known
as "Wiring". Therefore, in order to simulate an Arduino, I
needed a C/C++ compiler and I needed to re-implement Wiring. Oh, and that compiler needs to
run on iOS, integrate into my circuit simulation, handle
bad code such as infinite loops and bad pointers, and all
work within the sandbox (which means interpretation instead
of real execution). Like I said, it was a big request.

As tough as all that sounds, I still personally wanted the
feature. Way back in 2010, I started work.

I first looked around for small C++ compilers and
interpreters that I could
get to work on iOS. Sadly, the research was grim as no compiler
met all my requirements. Some would be nice and small but
only emit X86 code meaning I would have to write an X86 simulator. Others were so big and had so many dependencies
that I just gave up trying to get it to compile for iOS.

Fortunately in 2010, I was full of hubris and I figured now was the time in my life to write a C++ compiler. Oh how foolish...


## The Halcyon Days of 2010

Compilers and Interpreters are, in principle, very simple
programs. I'd written a bunch of interpreters at this point
in my career and even wrote a couple simple compilers for very
small languages. I knew that C++ was a complex language, at
least syntactically (with all its type declarations) but that
its semantics - its model of computation - was rather basic.
I considered myself a good C++ programmer and thought I knew
the language inside and out. How hard could it be?

Like I said, hubris.

And so I embarked on writing my C++ compiler in C#
(the langauge iCircuit is written in).
I started by writing a C compiler because C++ is a monster of a
language. Most Arduino programs only use C features (I thought),
so it seemed like a reasonable starting point.


### Writing the Parser

C is nice because you can actually parse it using
a *grammar definition* - a file that states the syntax of
the language. Grammars are great because you can use a code
generator to create a parser for your language from this 
definition. This means you can focus on the semantics of
the language and let the parser generator deal with the syntax.

Back in 1985 someone posted a [YACC grammar definition for C](https://www.lysator.liu.se/c/ANSI-C-grammar-y.html)
which served as my starting point.
I then used the [jay parser generator](https://www.cs.rit.edu/~ats/projects/lp/doc/jay/package-summary.html) (the same tools used to generate the mono C# parser)
as my parser generator. This tool takes the grammar definition
and spits out a nasty C# code to parse files. The
code is nasty because it's fast and eschews standard coding
conventions for performance.

The process of creating the parser went very smoothly thanks
to this. The real work involves creating C# syntax classes
that mirror the grammar. These classes form the Abstract
Syntax Tree (AST) of my compiler. You can see the results
of this work by [looking at my modified grammar](https://github.com/praeclarum/CLanguage/blob/master/CLanguage/Parser/CParser.jay).

I ended up writing a [variety of syntax classes](https://github.com/praeclarum/CLanguage/tree/master/CLanguage/Syntax) such as:

* `ForStatement` that captures the syntax of `for` loops
* `BinaryExpression` that handles most math operators such as `+` and `*`
* `Block` that captures a sequence of statements

And this is the point where I realized C was a bit more complex than I liked to think.

My syntax classes were filled with scary names like "abstract
specifiers", "type specifier", "type qualifier", "multi declarations", and so on. I knew C type declarations were nutty
but, oh my, they're a disaster.


### Definitions and Types

I was scared, but combatted that fear by writing a bunch of unit tests.
I figured, yes this problem is hard, but it's just big - there was an end in sight. Maximum effort would be needed and would, hopefully, be rewarded.

And so I pressed on. I just kept writing sample code after sample code until
I understood how my concept of the language related to this
grammar. After some time, I was able to wrestle all these "specifiers"
into more manageable objects.

The next step to writing a compiler is
discovering definitions in code. Variable definitions, function
definitions, type definitions, all that. C and C++ are a little
wild because you can declare things multiple times but you can
only define them once. While C++ is designed to be compiled using
only a single passs over the AST, I ended up writing a multi-pass
compiler with separate stages for declaration discovery, resolution,
and emission.

Once I found declarations and definitions, I needed to build
a type system. Thankfully, at first glance, C's types are very
basic. You have the machine types (ints and floats), 
structures, and, uh oh, pointers. The compiler would have to munge
around all the code and assemble and unify all these types.
I spent days and days just getting the integers right.

For example, we all know what this means:

```c
int foo;
```

And we all kinda know what this means:

```c
long foo;
```

But what does

```c
int long long foo;
```

mean? It's valid C. And how is that different from:

```c
long int long foo;
```

Not even the integers are simple in C...


### Emitting Executable Code

Now it's time for the compiler to earn its keep and emit
executable code. Most C compilers would emit X86 or ARM assembly.

However, there was no point in doing that as I can't natively
execute code on iOS. I decided to instead devise my own
byte code that would be easy to interpret.
I figured that if I controlled the compiler and the byte code
then I could arrange things to make the interpreter simple
and yet still expressive.

This was a bit of a gmable as it increased the number of
decisions I had to make. However, most byte codes I looked into
were very complex and I knew (I thought) I could keep my code
small and simple.

I settled on a stack-based virtual machine that is very similar
to how the Common Language Runtime (CLR) works. Every function
had a stack and computations were performed by pusing and popping
numbers to and from that stack. This is different from most real
machines like X86 that are *register-based* not stack-based.
Register-based machines scared me a bit because they reminded
me of very hard to read chapters of very hard to read books.
I knew how to implement them, but I lacked experience with them
and went with the simpler design. I would later regret this
decision...

Now that I had a semi-specified byte code and virtual machine,
it was only a matter of translating my syntax tree into byte
code. If you ignore performance, this is a trivial step for the compiler.
You can see, for example, how `if` statements get compiled by looking at the
`DoEmit` method of `IfStatement`:

```csharp
protected override void DoEmit (EmitContext ec)
{
    var falseLabel = ec.DefineLabel ();
    var endLabel = ec.DefineLabel ();

    Condition.Emit (ec);
    ec.EmitCastToBoolean (Condition.GetEvaluatedCType (ec));
    ec.Emit (OpCode.BranchIfFalse, falseLabel);

    TrueValue.Emit (ec);
    ec.Emit (OpCode.Jump, endLabel);

    ec.EmitLabel (falseLabel);
    FalseValue.Emit (ec);

    ec.EmitLabel (endLabel);
}
```

This code first emits the condition. It then emits a branch to one of two
blocks - either the main body (`TrueValue`) or the `else` body (`FalseValue`).

I stole this emit architecture from the mono compiler - always steal from the best.

Most of those early decisions paid off well and after a few weeks
of work I was able to compile and interpret the most basic of
Arduino programs.

It was the hardest I ever had to work to get a stupid LED to blink.


### Enough

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

Eight years is a long time and it's funny how memories distort. 
I was working on the feature set for iCircuit 2 and Arduino was first on that list.
I knew I had a compiler capable of making an LED blink, and my memory
told me the compiler was nearly done it just needed a *bit* more work.

I reopened the code and was pleasantly suprised at how much ground I had covered
so long ago. I also realized that my memory betrayed me - the compiler had some
serious defects (no support for strings as a prime example) and that it was
going to be a lot of work to finish it.

However, I decided that it was still useful and certainly allowed you to use
the majority of features in Arduino. 
So much so that I decided to release that compiler in **iCircuit 1.9** to 
finally provide the most requested feature for the app.
I built the Arduino component that interfaces with the compiler and simulator,
fixed a few bugs, and shipped it.

I was nervous, but very pleased to see that the 
Arduino component quickly became one of the most-used components in the app
and users seemed to love it.


### Back to Work

The enthusiasm for the Arduino component made me want to improve it
and I set about fixing its most glaring defects:

#### Numbers

All math in the 2010 compiler was done with 32-bit integers. This needed to be fixed
to support other integer sizes, unsigned math, and floating-point numbers. This
also means conversions (casts) betweeen all the types would need
to be supported. It's unglamorous and terrifyingly boring code to write
but it needed to be done.

I have since concluded that 95% of a C compiler's job is to convert between data types.

#### Pointers and Arrays

Pointers and array support exposed the most glaring defect in my virtual machine -
the use of a stack-based machine with separate memory spaces.

Unfortunately, C++ assumes a unified memory space - you can create pointers to
global data, heap data, local variables, even function arguments. Supporting separate
memory spaces becomes tricky because C++ (annoyingly) supports pointer arithmetic
and people can do terrible things by casting pointers to integers.

I decided to stop fighting and to unify my memory model. Before, the function call
stack was separate from the heap and kept its own private memory spaces for
arguments and local variables. I had to integrate that call stack onto the 
main memory stack by using a "frame pointer" register that remembers where in memory
each functions stack begins. I also had to emit relative addresses for function
locals and arguments that got offset by this frame pointer. Such a mess!

But the pain was again worth it as my compiler and virtual machine now could
handle pointers and arrays of any variety and acted as the user expects.

#### Classes

While most Arduino programs take advantage of C features, Arduino *libraries*
tend to take advantage of C++ features, namely, classes.

Everyone's favorite Arduino line:

```c++
Serial.println("Hello world!");
```

does not work on the 2010 compiler. It requires these C++ features:

1. Class definitions
2. Member method calls (with `this` pointers)
3. Function overloading

Each of these posed their own set of hurdles but were implemented, because,
as it turns out, people love printing to the console and I absolutely
*had* to get it working.

I'm very excited to say that Version 1.9.1 of iCircuit shipped with these improvements.


## The Future

While I love my compiler, I am still overwhelmed by the features it's missing. Still up for grabs are:

1. `#define`s with arguments
2. Templates
3. Constructors and destructors
4. Inheritance and virtual methods
5. Editor integration (to provide code completion)

and I'm sure a million other things if I kept a complete list.

But the truth is, I'm just going to keep listening to users and improving the parts I can.


