---
layout: post
title: ".NET Vector Performance"
thumbnail: "/images/2019/vectorperf/core.svg"
tags: article
---

**TLDR;** `System.Numerics.Vector4` and friends
give good performance across platforms -
especially for .NET Core apps. For iOS and other mono
apps, it's best to make your own simple vector types.
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

* `Microsoft.Xna.Framework.Vector2` (floats)
* `OpenTK.Vector2` (floats)

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
the normal JIT so I don't include it.

**mono 6.4.0.198** can be run in a variety of ways.
The default JIT is, well..., not fast so I don't include it here.
But you can run mono with the `--llvm` command line
and get a much faster JIT. Mono also supports AOT
with LLVM 
(precompile with `--llvm --aot`)
and that's also included in my analysis.

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

    Sadly, .NET Standard doesn't care about precision
    and does not provide a `Vector4d` type - so those results
    are missing.

## The Results


