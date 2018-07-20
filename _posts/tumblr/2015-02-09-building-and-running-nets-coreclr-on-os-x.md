---
layout: post
title:  "Building and Running .NET's CoreCLR on OS X"
date:   2015-02-09 17:58:00 GMT
redirect_from:
  - /post/110552954728
  - /post/110552954728/building-and-running-nets-coreclr-on-os-x
---



What you heard is true, [Microsoft has open sourced the CLR](http://blogs.msdn.com/b/dotnet/archive/2015/02/03/coreclr-is-now-open-source.aspx) and it runs on more than just Windows. They didn’t just dump some ZIP file on an FTP; no, we have *another* fully functioning, easy to compile, and easy to contribute to CLR hosted on everyone’s favorite file share. Microsoft has even gone so far as to setup a [two way mirror](http://) with GitHub so that their internal systems stay in sync with what we see. Color me impressed.

Their initial release works on Windows and Ubuntu. That’s nice - definitely a good start - but I’m a Mac guy. Microsoft said they’re planning on OS X support in the future but haven’t gotten around to it just yet. I figured I would take a look at the CLR in some months when they had the chance.

Of course, once things go open source, they tend to get out of hand.

Enter systems programmer extraordinaire [Geoff Norton](https://twitter.com/geoffnorton). Geoff knows everything there is to know about the CLR. He worked for years on [Mono](http://www.mono-project.com) - the original open source .NET - and has personal experience bringing it to OS X.

Well, if you did the trick once, you can do it again, right?

> Initial OSX support was just merged into CoreCLR https://t.co/CH1BWhWdyl@DotNet— Geoff Norton (@geoffnorton) February 7, 2015


<script async="" src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


> I believe that makes me the person who did bring up 64-bit OS X for all open source CLRs. Funny that.— Geoff Norton (@geoffnorton) February 7, 2015


<script async="" src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


It all started when Geoff “[had some time and made mac work](https://github.com/dotnet/coreclr/pull/105)”. Some time… Scanning through his PR, we can see the time went to:

* Patching the build system to teach it that Linux isn’t the only Unix out there
* Patching a lot of assembler directives as Mac’s `as` seems more strict than Ubuntu’s
* Rewriting bits of high performance assembly [in *bytes* (not opcodes)](https://github.com/dotnet/coreclr/pull/117#issuecomment-73343848) as Mac’s `as` refused to generate the desired instructions.
* Tons of spot unicode changes to appease the LLVM gods

In all, it looks like a lot of high precision and tedious work and I want to thank Geoff for doing it.

It’s not perfect yet. I had to spend an hour with Geoff just to get Hello World printing and, as I write this, there are more bugs. But open source wins out in the end. I went to bed with a bug and woke up [with its fix](https://github.com/dotnet/coreclr/pull/119).

But enough preamble! Let’s get going: let’s build .NET, write a bit of code, and then run it.


## 1. Get the Code


This is the easy part, the [CoreCLR is a git repo on github](https://github.com/dotnet/coreclr). You can fork that or grab Microsoft’s directly:

```csharp
git clone git@github.com:dotnet/coreclr.git
```


You now have more than 2 million lines of sophisticated code (C#, C++, and ASM) for a sophistacated platform designed to [make programming easy](https://github.com/dotnet/coreclr/blob/master/Documentation/intro-to-clr.md). I recommend a bottle of wine and a nice grep tool. But before then…


## 2. Get the Build Tools


[CMake](http://www.cmake.org) is used to build the Core CLR. I recommend installing it with [Homebrew](http://brew.sh):

```csharp
brew install cmake
```


In no time, you will have a hot little build tool.


## 3. Build It!


Yeah, seriously, it’s this easy:

```csharp
./build.sh amd64 debug
```


Only **amd64** is supported (which is fine, Apple dumped 32-bit like a hot potato), and we choose to do a **debug** build because it’s early days.

You should see `Detected OSX x86_64` and then lots of colors.

Just watch the pretty colors go by for a few minutes. And don’t worry about those red lines; CMake decided, somehow, that red is a good indicator of success.

If you have that bottle of wine, now would be a good time to get some crackers or perhaps a vegetable platter…


## 4. Admire It


When CMake has decided that you are ready, it will greet you with an inoccuous message:

```csharp
Repo successfully built.
Product binaries are available at /Users/fak/Projects/coreclr/binaries/Product/amd64/debug
```


We’ve waited nearly 15 years for this repo to successfully build, and now it has. My hat’s off to Microsoft’s CLR team that took the time to package this build so well. (I’ve downloaded single C files that were harder to build than this thing.)

Anyway, it has placed itself in a directory. Let’s go see what we got:

```csharp
$ cd binaries/Product/amd64/debug
$ ls -al
total 46032
drwxr-xr-x  5 fak  staff       170 Feb  7 11:47 .
drwxr-xr-x  4 fak  staff       136 Feb  7 12:25 ..
-rwxr-xr-x  1 fak  staff     49836 Feb  7 11:47 corerun
-rwxr-xr-x  1 fak  staff  23503712 Feb  7 11:47 libcoreclr.dylib
-rwxr-xr-x  1 fak  staff      4176 Feb  7 11:47 libmscordaccore.dylib
```


Those magic colors produced 3 files:


### corerun


**corerun** is a little driver that takes a managed assembly, initializes the CLR, then executes that assembly. It is the quivalent of the **mono** command but has a way worse name. This is the entrypoint for every managed app that you want to run on Mac.

We’ll look at its usage soon.


### libmscordaccore.dylib


What would a code base be without an appendix? Something that doesn’t really belong, but someone thought would be really neat to include?

**libmscordaccore** is that. I’ve heard that it “helps debugging”, but at 4KB, I have my doubts. No, this is just an ugly wart growing off an otherwise prefectly packaged piece of software. Best ignore it.


### libcoreclr.dylib


This is it folks. The big daddy, the heavyweight, the superstar, the high muckamuckstar, the big enchilada.

**libcoreclr** is three things - a damned good garbage collector, a JIT, a type and metadata system, and a bucket of platform abstractions. OK, it’s a lot of things – all packed into one super sexy dylib.

Remember those 100 MB+ downloads of .NET? Remember how long they used to take to install? Remember how once you upgraded .NET on the machine, all apps had to use it?

No more. No more install. No more DLL hell. No more deplyment hell. You want .NET? Here is a single file (and its appendix) that can be deployed with **rsync**. Here are all the important parts of .NET in one library weighing in at 24 MB. Not since Silverlight has the CLR been packaged so nicely (and, AFAIK, this is a much more sophisticated CLR than Silverlight’s).

Take a sip of that wine.

You should now save [The Book of the Runtime](https://github.com/praeclarum/coreclr/blob/master/Documentation/intro-to-clr.md) to your favorite Read-it-Later service. When you sober up from this experience, that document will be waiting for you - desperate to impart its decades of knowledge and wisdom on your “I’m too busy to study a runtime” adaled brain.


## 5. You’re Going to Need a String Class…


You might notice one thing missing from those three files: mscorlib.dll. That is to say, “where’s my String class?”

You see, while **libcoreclr** knows all about objects and types and how to manage them and keep them happy, it doesn’t actually have any. It’s heard rumors of types frollicking about, but it doesn’t have any to play with itself. It’s sad really.

While the CoreCLR repository contains all the code to mscorlib, it’s all written in C# and the CoreCLR doesn’t ship with a C# compiler! Because there’s no compiler, the build system can’t compile the standard library. It’s funny, in a not really funny kind of way.

The .NET team [recommends using a Windows machine](https://github.com/dotnet/coreclr/wiki/Building-and-Running-CoreCLR-on-Linux) to build mscorlib with the command:

```csharp
>build.cmd unixmscorlib
```


But, um, yeah, that’s cheating.

Enter, again, Geoff Norton. He has built that code and released it. [Download mscorlib.dll from Geoff’s Dropbox](https://www.dropbox.com/s/zvl5tsj6peh12km/mscorlib.dll?dl=0).

Just drop it in with the three others:

```csharp
$ ls -al
total 51776
drwxr-xr-x  6 fak  staff       204 Feb  7 12:34 .
drwxr-xr-x  4 fak  staff       136 Feb  7 12:25 ..
-rwxr-xr-x  1 fak  staff     49836 Feb  7 11:47 corerun
-rwxr-xr-x  1 fak  staff  23503712 Feb  7 11:47 libcoreclr.dylib
-rwxr-xr-x  1 fak  staff      4176 Feb  7 11:47 libmscordaccore.dylib
-rw-r--r--@ 1 fak  staff   2937856 Feb  7 12:34 mscorlib.dll
```


Congratulations, these four files combine to form a **fully functioning implementation of .NET on OS X**, courtesy of Microsoft and Mr. Norton.

(Some day we will have to contribute a script to CoreCLR that builds mscorlib with Mono’s compiler.)

I know you’re begging to run something, but let’s take a quick detour to show how to compile C# code using Mono on OS X.


## 6. Write Some Code


While we now have a fully functioning .NET, we need to give it something to run. If you have some assemblies lying around, they will do nicely, but let’s exam compiling code on OS X. We could use Microsoft’s compiler on Windows then just copy the assemblies over to OS X, but that’s cheating.

We will use [Mono’s free C# compiler](https://github.com/mono/mono/tree/master/mcs/mcs). Simply [install Mono](http://www.mono-project.com/download/) and it will be in your PATH.

Well you know what this is going to be… fire up your favorite editor and pump it out, `HelloWorld.cs`:

```csharp
using System;

public class HelloWorld
{
    public static int Main (string[] args)
    {
        Console.WriteLine ("Hello, world! From CoreCLR.");
        return 0;
    }   
}
```


Now compile it:

```csharp
$ dmcs -nostdlib -r:mscorlib.dll HelloWorld.cs
```


The `-nostdlib` option tells the compiler not to automatically reference any of its known standard libraries (not just mscorlib, but others too). In this case, we’re just playing things safe by explicitely referencing whatever assemblies we need by hand.

We then reference our mscorlib with `-r:mscorlib.dll`. This is necessary because of the previous option.

Indeed we can see that the compiler did its job and we have an assembly all ready for execution:

```csharp
$ ls -al *.exe
-rwxr-xr-x  1 fak  staff  3072 Feb  7 12:48 HelloWorld.exe
```


(Yes, we just used an open source project written as a free alternative to a closed source product to bootstrap said product when it became open source. The times we live in…)


## 7. Run That Puppy


```csharp
$ ./corerun -c . HelloWorld.exe
```


```csharp
Hello, world! From CoreCLR.
```


Fantastic.

Sip of wine.

We use `corerun` to begin executing .NET code. The `-c` argument is used to point it at the working runtime (the two dylibs and mscorlib). Since I’m doing everything out of one directory, I just point it at `.` (pwd).

That’s it! We have now have a fully functioning toolchain to compile and execute .NET code on sweet sweet OS X.


## 8. Contribute Back


Now let me warn you: this port to OS X is literally days old and contains known bugs. You cannot and should not release software using it.

If you are a systems engineer - or want to learn what it takes to be one - dive into that code. Try running your app, note the crash, and then load `lldb`. Chances are you will be lost, but you will be lost in a fantasic place full of mystery and possibility.

Then come hang out at [https://gitter.im/dotnet/coreclr](https://gitter.im/dotnet/coreclr). We’ll be chatting about all things coreclr and especially how to make the stupid thing stop crashing. ;-)


## Thoughts on the Future


The CoreCLR is an oddity in my mind - I don’t know quite where to file it away. 

We have had a high quality open source implementation of .NET on OS X for years now. I even make my living [writing](http://icircuitapp.com)[apps](http://calca.io) using it. I have spent the last 5 years learning it: its quirks and its powers. In addition to the technology, there is an amazing developer community that has grown up and matured around it for more than 10 years.

Microsoft can’t just drop a bunch of code on GitHub and expect a community to grow up around it (as nicely packaged as it is). Indeed, I’m not sure they want one to. Managing a community is a hell of a lot harder than managing pull requests. Higher management at Microsoft seems too concerned with Azure to care what non-Fortune 500 companies are doing with their wares. And, to be honest, that’s fine.

In the short term, Microsoft will continue to contribute to the CLR with their incredible programming and testing skills - we will marvel as they make precision changes to complex C++ code. The open source community will continue to pop up when they break something in our precious platforms or we decide that it’s time to do something novel and incredible with this little gem.

That is to say, for now, this is still *Microsoft’s* CLR. I am curious to see if it ever becomes *ours*.
