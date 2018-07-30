---
layout: post
title:  "Quantifying reflection's slowdown in sqlite-net"
date:   2010-11-14 15:51:00 GMT
redirect_from:
  - /post/1572668275
  - /post/1572668275/quantifying-reflections-slowdown-in-sqlite-net
tags: article
---



Yesterday I posted R64 of [sqlite-net](http://code.google.com/p/sqlite-net/) which included some performance upgrades thanks to [Joe Feser](http://twitter.com/#!/joefeser).

The tweet spread and we were asked how this compares to native performance of Sqlite. I had no answer because I've been drinking the MonoTouch kool-aid for awhile now and don't even think about native performance. But the questions did have me wondering.

I know there is nothing I can do about the overhead of a managed runtime like .NET over native code, but I did wonder how much overhead my use of Reflection introduces. To keep the use of sqlite-net simple (no external tool requirements, no on the fly code compilation), I rely on reflection to read and write values from properties. That design probably won't change since I love the ease of use of the library, but I do think its time to measure the overhead.

I wrote a little test app that serializes and deserializes 30,000 objects (most of whom's data is textual). The app performs those operations in three ways:

1. Using .NET's built-in BinaryFormatter and the [Serializable] attribute on the class.
2. Using a custom function that uses a BinaryReader/Writer that calls the appropriate Read/Write functions and sets the properties on the objects directly.
3. Using BinaryReader/Writer again but using generic reflection code that is very similar to the code that sqlite-net uses to read data from the database.

Through this, I can compare the performance of the reflection code (#3) vs the non-reflection code (#2). The use of BinaryFormatter is also put in as a control.

I ran the code 3 times on my iPhone 3GS using a Release build from MonoTouch. It writes approximately 7 MB files for each method and then reads those files back. Here are the average results:

* BinaryFormatter Serialize = **10.74 s**
* Explicit BinaryWriter = **1.30 s**
* Reflection BinaryWriter = **7.79 s**

* BinaryFormatter Deserialize = **50.38 s**
* Explicit BinaryReader = **2.93 s**
* Reflection BinaryReader = **11.21 s**

Two things are immediately visible in this data: BinaryFormatter is slow, and the reflection code is much slower than the non-reflection code.

(I have made the [Google Spreadsheet](http://spreadsheets.google.com/ccc?key=0AnbISxDzAVhOdHd4c1V1Sm9uNldhQ1hreVlJczVBdkE&hl=en) with all the data publicly available.)

Reflection introduced a slowdown of **3.8x** for reads, and a slowdown of **5.97x** for writes. I knew that the reflection code would be slower, but it's really good to now have that slowdown quantified.

And those are not small numbers! I was hoping for 50% slowdowns, but I guess that was too much wishful thinking.

Given the static compilation limitation of MonoTouch, the only ways to improve these performance number are to: (1) Force the user to hand-write the serialization code, or (2) Create a tool that generates that code automatically and integrate it into the build process. #1 seems like a really bad idea to me, so that leaves #2.

A somewhat related side note: I have finally looked at the Entity Framework CTP4 - specifically their code-first work. They have the same goals of sqlite-net and applaud their effort. I am thinking now of writing a little database that has the intelligence and interface of EF4's code first but that uses a .NET database. That way, you could share code between your iOS, WinPhone, Android devices and your server. A nice thought...
