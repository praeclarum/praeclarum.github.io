---
layout: post
title: ".NET Vector Performance"
thumbnail: "/images/2019/vectorperf/core.svg"
tags: article
---

**TLDR;** I wanted to know what the fastest
vector types were on .NET. Turns out,
performance varies wildly across platforms.
`System.Numerics.Vector4` and friends
give good performance overall,
especially for .NET Core apps, while homemade vector
types do not get auto-vectorized.
Avoid `Vector<T>` like the plague.
Oh, and iPhone 11s are stupid fast.


## .NET and Vectors

For years .NET didn't ship with any vector types
(small fixed-size arrays of primitive types)
and we developers would write our own
because (a) it isn't all that hard and
(b) the OS usually provided adequate types.

Even if platform types were adequate,
they weren't useful when writing cross-platform apps.
To help with this problem, the `System.Numerics.Vector`
types were added so we cross-platform devs
could share code between platforms
and with one another.
But, before those types were added, 
we all created our own vector types.
Many, many types.

You might be thinking, "Great! More choices are good!".
But you're wrong. More choices are bad.

For instance, when I'm programming on iOS,
I have my choice between all of these,
essentially the same, types:

#### .NET Standard Types

* `System.Drawing.PointF` (floats)
* `System.Numerics.Vector2` (floats)
* `System.Numerics.Vector<T>` (floats or doubles)

Works cross-platform, but will require conversion
when interfacing with the OS (rendering).
The System.Numerics types are supposed to
access the underlying SIMD hardware of the machine
for speed.

#### OS Types

* `CoreGraphics.CGPoint` (doubles)
* `SceneKit.SCNVector3` (floats or doubles, differs between iOS and macOS)

No conversions needed when rendering and
they probably (hopefully?) have been optimized.
Unfortunately, your code is no longer cross-platform.

#### Library Types

* `OpenTK.Vector2` (floats)
* `Microsoft.Xna.Framework.Vector2` (floats)
* `NGraphics.Point` (doubles)
* `SkiaSharp.SKPoint` (floats)

So many...
These are needed when accessing specialized
libraries but are also cross-platform.
OpenTK, in particular, has been my favorite
for years since they provide a full suite
of vector and other linear algebra types.

#### My Own Types

* `Frank.Vector2` (whatever, I'm in control!)

Of course I'm going to make my own type!

**Pros:** I can base it on my needs instead
of adapting to someone else's idea of how
a vector should work. Also, they're cross platform.

**Cons:** They will never be as fast as SIMD vectors,
I will have to keep converting between OS types and my own,
and they are not shareable with other devs.

## A Need for Speed

When faced with such a variety of choices
and trade-offs, it's hard to make a decision.
We need a tie-breaker. In the grand tradition
of 3D programming, let's make our decision
based on which one is fastest!

This is also an opportunity to check our
assumptions. For instance, the `System.Numerics`
types are supposed to be the most heavily
optimized - but are they? Also, 
does the platform influence the speed of these types?

The only way to find out is to write some micro-benchmarks
and try them out. I've been wanting to do this for years, let's
get going!

### The Competitors

I narrowed my study down to just cross-platform types
since they're the most useful to me.
I also decided to test only 4D 32-bit floating-point vectors since
that is the most common SIMD size between the platforms.

* **`MyVector4`** is the simplest vector type I could write.
It's included as a baseline for all the other types.

* **`OpenTK.Vector4`** is my long-time favorite
since it filled the gap before `System.Numerics` came along.
I have high hopes for this one.

* **`System.Numerics.Vector4`** is the presumed
champion since lots of smart people have worked on it
and because its documentation tantalizes you with
the possibility of hardware acceleration.

* **`System.Numerics.Vector<T>`** is what
we're all supposed to use when we are not using
32-bit floating point data. This is important to
me because a lot of my apps use 64-bit data for
precision. It is also supposed to be hardware accelerated.

* **`float[]`** is used to compare the performance of not using
vector types at all. This is not preferable from a coding
standpoint since you lose the vector abstraction;
but it is well-known in graphics programming that the CPU
doesn't care about your abstractions and just wants the data
as fast as possible. Array serves that purpose well and
is included as a baseline.


### The Platforms

.NET runs on everything with a CPU (and lots of RAM)
so we have a lot of platforms to choose from when
measuring performance.

The two most popular .NET runtimes are .NET Core and mono
(Ima ignore classic .NET framework).

**.NET Core 3.0.100** is newly released and contains the impressive
RyuJIT execution engine. You can run apps with the JIT or
using the "Ready to Run" ahead-of-time compiler.
In my tests, the AOT didn't perform any better than
the normal JIT so I didn't include it.

**mono 6.4.0.198** can be run in a variety of ways.
The default JIT is, well..., not fast so I didn't include it here.
But you can run mono with the `--llvm` command line
and get a much faster JIT. Mono also supports AOT
with LLVM 
(precompile with `--llvm --aot`)
so that's also included in my analysis.

**Xamarin.iOS 13.2.0.42** is the mono LLVM AOT running on ARM64 chips.
I make my living writing apps so this is the most important
test for me.


### The Machines

The .NET Core and mono tests are run on my
**3.0 GHz Xeon iMac Pro** from 2017
running macOS 10.14.6. It's a beast.

The Xamarin.iOS tests are run on my new
**2.65 GHz A13 iPhone 11 Pro** from 2019
running iOS 13.1.something. It's green.

The Mac should be a lot faster than the phone
since it's plugged into the main power grid,
has decades of optimizations,
and is clocked faster than the iPhone.
It also weighs a lot more than the phone.
Heavier things should be faster, right?

### The Competition

Four scenarios have been devised to try to capture
the relative performance of the vectors:

* **Multiply by a scalar:**
Given an array (length *N*) of 4D vectors, how long does it take
to multiply all the elements of the vectors by a value?
This is a very common operation in 2D graphics
and is a simple place to begin our comparisons.
Computation will require `4*N` floating-point multiplies.

* **Normalize Vectors:**
Given an array (length *N*) of 4D vectors, how long does it take
to normalize all of those vectors (make their magnitude equal to `1.0`
without changing their direction)?
Computation will require `4*N` floating-point multiplies,
`3*N` adds, `N` square roots, and `4*N` divides
(or `N` divides, and `4*N` additional multiplies).
This is a very common operation in 3D graphics
and should be optimized by any self-respecting vector library.

* **Add vectors:**
Given two arrays (length *N*) of 4D vectors, how long does it take
to add each element of these arrays together?
Computation will require `4*N` floating-point additions
but a lot more memory reads than the scalar test requires.

* **Add 64-bit vectors:**
This is the same as the previous add test but
using 64-bit values instead of 32-bit.
While most graphics renderers use 32-bit data,
most code apps (of the sort I write) use 64-bit math
to maintain accurate results. All of my apps internally
use 64-bit math so these tests are important.

    This test was limited to 2D vectors since
    ARM64's SIMD register can only hold 2 64-bit values.

    Sadly, .NET Standard doesn't care about precision
    and does not provide a `Vector4d` type - so those results
    are missing.

`N = 16384` in the results below.

## The Results

(Lower is better.)


### .NET Core

<img src="/images/2019/vectorperf/core.svg" width="100%" alt="Chart showing that System.Numerics is very fast">

#### Observations

* **`System.Numerics.Vector4` is crazy fast!**
Bravo to the
.NET Core team for their work here. It is over 6x faster than
my vector and OpenTK's (and even faster when normalizing).

* **`System.Numerics.Vector<T>` takes 2x as long as Vector4**
This is understandable given that, on a Xeon processor,
`Vector<T>` does twice as much work as `Vector4`.
This is because it holds and operates on twice the
number of values; it holds 8 32-bit values instead of
`Vector4`'s 4. But even with this additional effort,
it still outperforms custom vector types.

* **Arrays are faster than using custom vector types**
Sadly, the graphics devs were right. Throw away your abstractions
and embrace off-by-one errors if you cannot use the optimized
`Vector` types. Fortunately, they're only marginally faster,
so maybe don't throw away your abstractions.

* **My custom vector is reliably slightly faster than OpenTK's**
I have no idea why. Optimizers are fickle. My guess is that
there is some attribute attached to the OpenTK type that prevents
some optimizations.

#### Conclusions

If you're only targeting .NET Core, you should absolutely
make use of the `System.Numeric.Vector` types.

.NET Core's vector optimizations are fast, but only if using types that they have pre-chosen to receive those optimizations.
It would be nice if the optimizer could detect that
other classes have the same shape as `Vector4` and optimize them.

But now the big question: does this performance translate
to the other platforms? Let's compare to Xamarin.iOS,
a very different platform from .NET Core,
to find out.


### Xamarin.iOS

<img src="/images/2019/vectorperf/ios.svg" width="100%" alt="Chart showing that System.Numerics is no longer fast and custom types are">

Oh how things have changed. I have so many thoughts...

#### Observations

* **The iPhone is crazy fast!**
Did you notice I kept the same vertical scale as I used for the Mac?
Yes, the iPhone, that tiny little device in your pocket, is as fast
as a big desktop computer. The times we live in...

    This isn't just a testament to the iPhone, it's a testament
    to the great work done by the Xamarin team to create
    such a fast runtime. Props also go out to the LLVM team
    for creating such a wonder.
    
    Code that operates on my vector type and the OpenTK vector
    type runs faster on my phone than on my computer.
    It's still sinking in...

* **The performance gap between my vector and OpenTK's has widened**
This is still a mystery to me. From the data, we can observe
that my code got optimized while the OpenTK code got less so.
This is definitely a topic worth pursuing. I would need to
analyze the actual code generated to see why such a difference
exists and then do further analysis to see what C# definitions
caused that difference.

    I didn't do any of that work. So, for now, let's just say
    that simplicity wins out since my type is stupid simple.

* **The optimized `System.Numerics.Vector4` is slower than my custom vector**
This is a very surprising (and sad) result. All I can say is that
the optimizer took a liking to my code for some reason.

    LLVM is a vector-optimizing compiler. If it sees an opportunity
    to turn some code into a vector operation, it will.
    This is a powerful optimization because it means that it's
    not limited to a certain set of types. Instead, it figures
    things out on its own and optimizes as it sees fit.

    So why did it optimize my code and not `Vector4`,
    presumably with the mono runtime helping it out?
    I have no idea and welcome your thoughts on Twitter.

* **`System.Numerics.Vector<T>` is broke**
I don't know any other way to say it. It's performance
was so poor that it's literally off the chart.
It has the same size
as `Vector4` on ARM64 so you would expect it to perform as well.
But it doesn't.

    I can guess here. Value-type generics on mono AOT
    are um... special. They use a lot of tricks to share
    code and those tricks come at a performance cost.
    My guess is that code generator is not recognizing
    `Vector<T>` as a SIMD register and is instead
    compiling it as a complex generic value type
    with all the attached performance penalties.

### Conclusions

Wow things have changed since the .NET Core chart.
`Vector4` is no longer king. Instead, a simpleton upstart
has outperformed it for reasons I'm still guessing at.

Worse, the cross-platform SIMD register, `Vector<T>`
is not working as intended and must be avoided.

The iOS chart is so different from the .NET Core one,
that I think it's time we look at a middle ground.
Let's see how mono runs on the Mac.

## Mono LLVM

<img src="/images/2019/vectorperf/mono.svg" width="100%" alt="Mono LLVM Chart showing that mono is slower than .NET Core but Vector4 is fast again">

### Observations

* **Mono with LLVM isn't as fast as .NET Core**
LLVM is fast, but RyuJIT is faster!

* **`System.Numerics.Vector4` is back on top**
It's good to see that the x64 mono is able to optimize
the SIMD vector.

    Except when it can't. Oddly, it wasn't able to
    optimize the normalization operation as .NET Core was able to.

* **`System.Numerics.Vector<T>` is sometimes fast, sometimes not**
`Vector<T>` is back to being its unreliable self. Sometimes
its as fast as `Vector4`, other times its as slow as custom types.

* **OpenTK is faster than my type**
Optimizers are so random. Who knows...


### Conclusions

I'm not sure if I feel more enlightened, or more confused.

One thing is becoming clear: the relative performance of
vector types depends on the platform.
Oh, and `Vector<T>` is terrible. That's two things.

In an attempt to bring some order to the chaos, let's
look at mono again, but this time let's let it
compile ahead-of-time.


## Mono LLVM AOT

<img src="/images/2019/vectorperf/monoaot.svg" width="100%" alt="Mono LLVM AOT Chart showing that nothing in software is predictable">

Of course everything changed, FML...

### Observations

* **My custom vector type is now faster than OpenTK**
What's the deal with optimizers? Why are they so fickle?
My vector got a big optimization thanks to the AOT
while OpenTK's vector did not.

* **`Vector4` didn't change much compared to non-AOT**
It's good to know that the JIT and the AOT are comparable.

* **`Vector<T>` becomes an even bigger train wreck**
It seems to suffer from the same bug we saw on iOS.




## Overall Conclusions

It's all random. Give up understanding how computers work.

*Cough*

OK. I'm back.

If you're writing cross-platform code, there is
no single set of vector types than you can be sure
are fast across all the platforms, sorry.

But I feel confident with the following advice:

* **Use `System.Numerics.Vector4` and friends** as they
are *usually* not the slowest and sometimes are the fastest types.
Also, they're cross-platform. What's not to love?

* **Avoid `System.Numerics.Vector<T>`** unless you
are constrained to one platform and you know it works there.
That is, don't use it in .NET Standard libraries.

* **For all other vector types, just use anything from nuget.**
Sometimes my type was faster, sometimes the OpenTK type was
faster. Which one was faster was hard to predict and depends on the platform.

    None of the .NET runtimes (except, maybe Xamarin.iOS?)
    seem capable of auto-vectorizing code. So it really doesn't
    matter which custom vector type you use.
    
    My only advice
    would be to keep the definition as simple as possible
    so that maybe, just maybe, it will get optimized.


## Data and Code

While the results felt random, they were reliably produced.

The raw performance numbers can be found in this Google Sheet:
[https://docs.google.com/spreadsheets/d/1IAT8ApgcugllmfvUy4ly61i1ZslaTGy-plqAGubI-kU](https://docs.google.com/spreadsheets/d/1IAT8ApgcugllmfvUy4ly61i1ZslaTGy-plqAGubI-kU)

