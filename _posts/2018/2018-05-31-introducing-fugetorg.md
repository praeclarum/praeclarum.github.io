---
layout: post
title:  "Introducing fuget.org"
date:   2018-05-31 17:50:19 GMT
redirect_from:
  - /post/174440517348
  - /post/174440517348/introducing-fugetorg
---



Have you ever wondered what exactly is in a nuget to see if it’s right for you? You read the description, you like the name, but, if you’re like me, you probably ended up in GitHub reading the source code to decide if you want to use the library.

I love nuget and the wonder of diversity of packages in it, but I didn’t love *browsing* it - it was just too much work to see what a library was really offering. I decided that I wanted (and needed) a better package browser streamlined for this exploration process.

Today, I’m pleased to announce [fuget.org](https://www.fuget.org) - a new site for browsing nuget packages. It is my best attempt to build a tool to help you both discover new packages and to dig in deep to learn them once found.

![image]({{ "/images/tumblr/174440517348_0.png" | absolute_url }})

[Fuget.org](https://www.fuget.org) shows you most of what nuget.org shows you, but adds these exciting features:


## Supported Framework List


Have you ever wondered if the library your using has been customized for a certain platform? Have you wondered if it will work on your platform at all?

This doubt is removed by displaying - in full technicolor - all the frameworks that the library supports.

![image]({{ "/images/tumblr/174440517348_1.png" | absolute_url }})

They’re color coded so you can see at a glance:

* Green libraries are .NET Standard and will work everywhere
* Dark blue libraries are platform specific
* Light blue libraries are for full .NET and Mono only
* Yellow libraries are old PCLs that we’re all trying to forget

This solves a general pet-peeve of mine of wanting to know exactly what code I’m getting for my platform. Usually, thankfully, most libraries are .NET Standard. But for those that aren’t, it’s important to know if it will work for you.

Related to this, it’s often enlightening to learn the differences between different APIs on different platforms. When you click a library, fuget.org reads its assemblies and allows you explore its API. You can use the framework buttons to see how those APIs differ between platforms. This can let you see, for example, how Xamarin.Essentials implements secure storage on iOS vs secure storage on Android.


## API Explorer


This is the big feature that I’m most excited about. All the classes in your assembly are browsable - you can drill down starting from framework, to assembly, to types.

When you click a type, its members and its documentation are displayed for you. Each member’s declaration is shown along with a short summary built from the XML docs library developers lovingly write. All of the types in the declaration are cross-linked throughout the site - even between packages - to help you explore the API.

There is even an API search - just start typing in the little box to find links to whatever types or members you’re interested in. When you click a result, the documentation for that item will be shown. If you click a type whose documentation is provided by docs.microsoft.com, then you will be redirected there instead.

If the library is open source, then a Code tab will appear that lets you browse the source of the library.


## API Diff


In the modern world of release many, release often library development, it’s sometimes hard to tell what’s changed between different versions of a library.

To solve that problem, [fuget.org](https://www.fuget.org) can automatically generate API diffs between any two versions of a nuget package.

![image]({{ "/images/tumblr/174440517348_2.png" | absolute_url }})

It looks at the public API - all the types and all their members - in both versions of the library and shows you what was added and removed through a cute little hyperlinked diff.

I recently used this to see how Xamarin.Forms 3.0 is different from 2.5. It’s fascinating to see how libraries have matured over time.


## Easy Entrypoints


The URLs of [fuget.org](https://www.fuget.org) mimic [nuget.org](https://www.nuget.org) so if you ever find yourself on nuget.org but wish you were on fuget.org, just change that “n” to an “f” and you’ll be all set.

There is also a package search at the top that gives you the same results as your favorite IDE would give you.

Lastly, the homepage remembers your most commonly visited packages and keeps a short list for you. This is all stored in your browser’s local storage and never transmitted to the server - so your obsession with JSON libraries will remain a secret.


## And More


I built this site as a tool for myself but am glad to finally share it with everyone. This thing has been a real work of love and I hope it helps you in your day to day work.


## Colophon


I built the site using ASP.NET Core on a Mac with Visual Studio for Mac using Razor pages. It is hosted on Azure.
