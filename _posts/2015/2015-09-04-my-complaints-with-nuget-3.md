---
layout: post
title:  "My Complaints with Nuget 3"
date:   2015-09-04 18:39:00 GMT
redirect_from:
  - /post/128347388533
  - /post/128347388533/my-complaints-with-nuget-3
tags:
---

**Warning** This article is garbage. Don't read it.

**Argument** Nuget 3 makes my life harder, all in the name of solving a problem I don't have.

**Lemma** And it doesn't solve any of the problems I currently have.

**Conclusion** @#$E%^&&^%! meh...

My complaint against nuget 3 comes from its added burden and complexity hefted onto library developers.


## Prelude


Let me start by putting my cards on the table: if it's hard for me to support your platform in my library, I'm not going to bother.

In my mind then, every effort towards improving nuget has to improve it from a library developer's perspective. If you make it easy for developers, nuget will be filled with awesome libraries that can run on the ridiculously large number of runtimes. The ecosystem and community grow and we all get back to our jobs of making fun of C++ and JavaScript programmers.

If, on the other hand, you make it hard, as has been done with nuget 3, you get a whopping "meh" from people like me and a o_O from the community.

Library developers start on a platform. I start on Mac or iOS. I have only ever started two libraries where I set out to make them cross-platform. The rest I made cross-platform either because it was trivial (start with a PCL, more on that later) or because I was willing to make the Herculean commitment to make it cross-platform.

I say **commitment** because anyone can create a library once - a nuget (even a nuget 3) package is a tolerable time investment. What's not tolerable is creating build scripts and build servers that can compile and package everything every time I make a minor code change. Then getting those build bots configured in a way that the community can use them? Forget about it. (I don't mention any of the commercial build services because it's hard to justify monetary investment in OSS projects. I don't mention any of the free build services because they don't support my kind of builds which usually involve Xamarin.)

Back to platforms. Now, I've started a new library on a platform.

In the bad old days before PCLs, to release the library, I would have to make a bunch of junk projects for each and every fragment of .NET, all to convince msbuild to make me a bunch of binaries. This is just a silly assortment of meaningless names - Windows RT, Windows Phone, Windows PCL, Windows UWP, blah, blah, blah. 1 Library turns into *N* Projects.

(Personally, I see this as a major design flaw of msbuild. Imagine how different the .NET ecosystem would be if msbuild was actually a Common Language tool that could handle sources from multiple languages, imagine if it could output binaries not tied to a single platform, but "fat binaries" that just worked. Imagine if it was a build bot and not some CLI app from 1970. This is a tool I've written for myself a couple times when I was in my deepest throws of nuget and .NET cross-platform depression. Never released any version of them cause they play hell with the IDEs, but man it bothers me that no one else sees the project system to be one of .NET's major flaws (more on that later).)

Thankfully, PCLs came and saved the day. 1 Library remains *1* Project. I could ignore .NET fragmentation if I just picked one of the supersets. This means that the majority of my libraries and code could now be shared without creating a hundred meaningless project titles and build scripts. I even write my apps using PCLs even when I don't care about cross-platform. I do it because I might want to take that code and open source it. This is how I've always worked - I see a chunk of my app that I think others could benefit from, then I open source that bit.

With PCLs, open sourcing a library became trivial. I write a terrible XML file, I *don't* have to create any new projects, and I just put nuget in my Makefile. Done. (And sorry that I'm conflating "Open Source" with "nuget", but most .NET devs won't even blink at a lib unless it's on nuget.)

Of course, the necessarily platform-specific bits would have to be shaken out into their own projects. It's not a perfect system, but it's manageable. 1 Library turns into *M* Projects where *M* is the number of platforms I actually care about (it's not the multitude of .NET fragments). This isn't like a PCL where I want to run everywhere - this is a platform specific lib and I take on all the effort and commitment that it implies. (I wish this effort was smaller, but the IDEs don't seem to care about library authors.)


## Enough Prelude, What's Wrong with Nuget 3?


Nuget 3 was an opportunity to fix the few things wrong with nuget and make the world a better place. Nuget 2 has a couple design mistakes that I would love to see corrected in a new version:


1. It has no concept of "families" of libraries so platform specific libs - or libs that have been partitioned on one axis or another - each act like standalone libraries. Look at the hilarity of the FunScript libraries. Look at the FSharp Data providers. Or, if you have a sufficiently stiff drink nearby, look at the numerous ASP.NET libraries. I have no idea what any of them are or how they're related. Nuget has a very simple dependency graph that concerns itself only with binary dependencies, not conceptual. That's to say, it works fine for machines, but is a long way from humane. If libraries could join families - the catalog could be cleaned up and lib devs would feel safer partitioning their libs.

2. That partitioning I mentioned? Libraries get split up for millions of reasons. Perhaps it's due to platform. Perhaps a large feature is split out. Perhaps the lib developer loves the modern world of 1 class per library. Whatever their reasons, almost all large nugets are partitioned. Unfortunately, nuget (and its UI) leave it up to the consumer to reason what those partition axes are and how they apply to a project. If these axes were first-class (reified, whatever), we could turn the catalog into well organized and friendly place for both lib developers and consumers. Instead, it's just an FTP directory with a bunch of DLLs in it with a big sign: "You better RTFM"!

3. Even the the simple dependency system is broken. If I add library A that depends on B, then remove A, I still have B lying around. This is just an embarrassing bug that should be fixed.

    OK, maybe it's unfair to judge nuget 3 on what it's *not*. But with its slow update cycle - seemingly tied to Visual Studio - it's hard not to regret missed opportunities.

2. Nuget 3 upheaves the entire ecosystem. Old nuget: PCLs + Platform Specific bits (finally we hit a panacea). New nuget: PCLs? (maybe? I honestly have no idea if I'm supposed to write PCLs anymore) + Platform stuff + CoreCLR. Wait what? CoreCLR? You mean that thing that still can't run Hello World yet? My nugets got torn to shreds to support that thing? I know it's the future, and it's an exciting future, but OMG we are a long way from there. You have introduced a new platform (that doesn't work) and said that nuget is now based off of it.

3. Seriously, are PCLs deprecated now? A running theme in my criticism is a lack of communication about *how to write libraries* in this new world. I know enough to know that nuget 3 has a complicated facility to resolve between PCLs and "dotnet" - so I guess PCLs still work? But am I supposed to stop making them? Should my cross-plat libraries be dotnet based or PCL? No one will stand up and answer that question without their own several-paragraph prelude. If "dotnet" is the future, it's one shrouded in mist.

4. I am so confused by DNX, DNVM, and that thing called project.json. I have no idea if these things are related to nuget 3, but they have the same scent. Let me repeat, **I have no idea** if this nuget 3 stuff has anything to do with those techs. I am so confused by buzzwords and cute project names and blog entries that I've completely lost the narrative. Those tools are supposedly how you run code on the CoreCLR (why oh why? why couldn't we just have a simple executive. Oh? Because web people environment variables? srsly?)

5. Or was it package.json? Confusion continues. Maybe next year we'll have purpose.json. And then the year after, promise.json. And then, no-seriously-use-this-project.yaml (haven't you all noticed yet that JSON is a terrible format for hand editing? XML is easier. YAML is easier. JavaScript is easier. TSON or any of the other *SONs are easier.).

6. Let's say I choose to embrace "dotnet". Well, I can't because Xamarin doesn't support it. This is a letter to Microsoft, so perhaps you don't care. But it's my main form of .NET consumption. If Xamarin doesn't support it, it might as well not exist. I can guarantee you I will actively ignore nuget 3 until Xamarin supports it.

7. Still hypothetically embracing "dotnet"... what is up with the manual dependencies? Breaking up the BCL is some sick joke. I was in denial for a long time, then I got angry, now it just makes me sad. There is no more stdlib. We get, what?, int and string? And now I have to import libraries for everything else? This partitioning may have some technical benefits, but I don't see them. It's just added effort for what? I guess I can now run newer versions of System.Collections and old versions of System.Text? In what world does someone need to do that? A reminder: users of .NET are on a platform - we may like to consume cross-plat libraries, but we *use* a specific platform. I use mono. It updates its libraries every year or so. It's an exciting time of year - retesting apps and making changes and filing bug reports. The thought of libraries now following their own independent release schedules just makes me shutter.

8. Whatever, I'm on the losing side of history for wanting a monolithic class library. So let's say I fall down into a well and my only way out is to solemnly commit to embracing "dotnet". I am still confused about their relationship to PCLs. Every time I hear someone discuss the resolution rules for nuget 3, I dream of my peaceful days back in that well. If I install Visual Studio 2015 Community edition (thanks so much for that btw!), and I create an additional project in parallel to my PCL project. Now I'm managing two project files instead of 1. One is classy and takes care of itself. The other has brain damage and I need to hand hold it and it's 100 dependencies. Or am I supposed to throw out the PCL?

9. Let's say my time out of the well has reformed me and the CoreCLR is actually a viable target. Well, nuget3's file format is still a terrible bastardization of something that used to be simple. We keep shoving more and more rules and features into this schema that the file is a mixture of configuration *and* convention. I keep mentioning the resolution rules for nuget 3. Where are they written down? Which binaries does XS or VS pick given the set of available platforms? There are blog posts that make rough English impressionist style drawings of this algorithm - but nothing definitive.

    What I really want is a matrix with "nuget platform" as one axis and "real platform" on another. Now, if I want a library that I know works on a given "real platform", then I merely have to look on the row and find which "nuget platforms" that corresponds to. Ideally, an organization with funding would maintain this matrix - Microsoft, the .NET Foundation, Xamarin, Mono, anyone. Except "the community". The .NET community is important, but since we don't get a say in nuget design decisions and since this matrix is becoming more and more complex with every nuget release, the people doing the damage should take responsibility.

10. I am sad that I desire such a matrix. Sad that .NET has fragmented so much that it's needed. But instead of nuget 3 coalescing that fragmentation, it just created more.


## Caveat


You may be reading this document and shaking your head "he just doesn't get it".

That is 100% possible. Maybe nuget 3 actually improves my life and I'm acting like an out of touch old codger.

But I guess that's my point too. If nuget 3 really is a fix for the fragmentation problem, then why is the present so gray and cloudy? Why are OSS library devs who have been doing this stuff for years so confused? For goodness sake, even Newtonsoft is confused and they are Microsoft's darling example.

Why isn't anyone shouting "PCLs are dead, all hail the Core CLR and it's 100 dependencies!"

Is nuget 3 ahead of its time, or simply the answer to the wrong question? Only time will tell I guess.
