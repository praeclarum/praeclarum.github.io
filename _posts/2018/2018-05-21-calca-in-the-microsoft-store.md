---
layout: post
title:  "Calca in the Microsoft Store"
date:   2018-05-21 19:43:11 GMT
redirect_from:
  - /post/174121951203
  - /post/174121951203/calca-in-the-microsoft-store
---



[Calca](https://calca.io) is my crazy symbolic math calculator/markdown editor designed specifically for mad scientists. Today, I am very pleased to announce that [Calca 1.5 is available in the Microsoft Store](https://www.microsoft.com/store/productId/9NHXZ5159N41)!

![image]({{ "/assets/tumblr/174121951203_0.png" | absolute_url }})

This is exciting for me on two fronts.

First, this is a great update to the Windows version of Calca that **includes plotting and high DPI support**. Plots make it easy to visualize functions and calculate derivatives while the high DPI support just makes the app look good. This version also includes a large number of fixes and I hope you love it!

Second, and the point of this blog post, is the fact that the app is finally available in the **Microsoft Store** making it super easy for all Windows 10 users to try it.

This is a change from my previous distribution method of hosting the app on my own store. I never liked that approach for a variety of reasons - updates were hard, visibility is hard, users had to trust my payment processor, etc. etc. now that it’s in the Microsoft Store, I hope to reach more people and simplify the update process.

The trick was that I designed Calca for Windows to work on Windows XP and does not run on Microsoft’s UWP platform - previously a prerequisite to being in the store. The good news is that Microsoft is now allowing such apps in the store thanks to a program called **Desktop Bridge**.

If you’d like to hear more about it, please continue reading. Otherwise, you can go get a [free trial](https://www.microsoft.com/store/productId/9NHXZ5159N41) to see if it’s for you.


## All Hail the Desktop Bridge


Microsoft heard all us Win32 programmers begging to be the in Microsoft Store and launched “Project Centennial” - a great program with an insulting name. So they renamed it “Desktop Bridge” and all the world’s programmer’s rejoiced. With The Bridge, you can get your .NET WinForms apps into the Microsoft Store and onto millions of Windows 10 machines.

I’ll be honest, I’m still not sure what The Bridge ***is***. The good news is, you can trust it without knowing what it is because it doesn’t modify your app. Instead, it wraps up your app’s executable and all its support files into a standard APPX package.

It’s as easy as downloading the DesktopAppConverter from the store and running this command in PowerShell:

**DesktopAppConverter.exe -Installer *C:\LocationOfAppAndDependencies* -AppExecutable *App.exe* -Destination *C:\DesktopAppConverterOutput* -MakeAppx**

Well, that’s the general idea. Unfortunately, the documentation for the converter is a bit sparse and it took me some trial and error to learn the full set of arguments you need to pass it to work. Most of these arguments simply match what’s displayed in the Microsoft developer hub - however they have slightly different names creating a bit of confusion. Incase you ever find yourself doing this, here’s a little guide:

**-PackageDisplayName** is the name of your app in the store. “Calca” for me.

**-PackageName** is generated by Microsoft and is something like “1B4DF00D.Calca”

**-PackagePublisherDisplayName** is you, but it better match the name in the developer hub. For Calca, it’s “Krueger Systems, Inc.”

**-Publisher** this is the GUID that Microsoft lovingly calls you in bed. “CN=X0X0X0X0-X0X0-X0X0-X0X0-X0X0X0X0X0X0″

**-Version** is in class 3-dot form: “1.5.0.0″


## Papers Please


Unfortunately, Desktop Bridge apps are still not fully supported by the Developer Hub UI. Instead, you are going to have to fill out some web mail template forms, wait a few days, do some other banal task, wait a few days, sign another agreement, wait a few days, you get the idea.

You will also get warnings about your app being special (I know!) and how certification will take a week instead of the usual couple hours on the store. This process also involves more emails and more clicking. It’s fun.

Overall, the process went very smoothly.

I was hoping for a more streamlined experience. None of this process is hard, it’s just very bureaucratic. And it does leave me wondering how much effort updates will be... but let’s not think about that!


## A New Age for Updates


My biggest regret with selling Calca directly is that I didn’t establish a good update path for customers. My policy was “email me and I’ll send you an update for a year”. This policy is bad both for my customers (I’m making them use email!) and myself (now I have to read email!).

The net result of this was the Windows version of Calca was falling behind the Mac and iOS feature sets. Terrible!

Thankfully, now that the app is in a proper store, I can keep it up to date with iOS and Mac cousins.

That’s it! Thank you for reading and I hope you have fun doing some math!