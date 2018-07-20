---
layout: post
title:  "My Xamarin Studio F# Wishlist"
date:   2014-07-11 18:04:00 GMT
redirect_from:
  - /post/91470574883
  - /post/91470574883/my-xamarin-studio-f-wishlist
---



I am writing a large iOS and OS X app in F# and am totally digging the honeymoon phase. Not only is F# a crazy powerful language, but it has a great interactive code executer built right into the IDE. In all, it's a wonderful development experience.

(If you haven't seen it, check out my hastily recorded Interactive [F# Development in Xamarin Studio](http://www.screencast.com/users/praeclarum/folders/Default/media/d257a92b-ec14-41e4-abb4-fb5b85ff22c4) video.)

While F# in Xamarin Studio is great, it does lack a few features I miss from the C# experience. These include:

**Sorted Intellisense.** If you didn't know, you can hold the Shift key while upping and downing through the Intellisense to switch to an inheritance sorted list of members. This is not only useful, but critical to exploring new objects and APIs. It also lets you filter out all the junk that comes with base classes (I'm looking at you NSObject). I miss this feature so, so much (imagine creepy Anakin Skywalker saying that).

**Expand Selection using the AST.** Another important but often missed feature of Xamarin Studio's editor is the ability to expand your selection based on the language's syntax tree. If you have the text "fo|o + bar", then expand the selection, you get "[foo] + bar". Expand again, and you get "[foo + bar]". This is an amazing keyboard shortcut that services quick identifier selection, expression selection, method selection, and more.

<strike>
  <strong>Rename Refactoring.</strong> You know, you don't think of it much, but when someone takes away your rename refactoring, you all of a sudden are back in the 1990's coding in Turbo Pascal with your crappy IBM PS/2 debating whether code clarity is worth the trouble...</strike>

Thanks to [Don Syme for setting me straight](https://twitter.com/dsyme/status/487684270048423937): **Xamarin Studio supports Rename** and it works great (even across projects). I'm going to meditate a bit on how it's possible to be completely blind to a feature.

**Search for Definition.** MonoDevelop used to have a great tool for jumping to any definition in your code. Just hit Cmd + . and you get a great search window. Xamarin Studio screwed this up a bit by putting the window in your peripheral vision and limiting the results to just a few items with not enough info to distinguish them. But I digress. Even in its not-very-useful state, I still wish I could search for and jump to definitions in F# code more easily.

**NUnit IDE Integration.** Xamarin Studio has shortcut "test bubbles" in C# code when using NUnit. These bubbles are right in the editor with the code so it makes running and debugging tests a two click task. While F# of course supports NUnit testing, the editor doesn't care. Too bad.

**And then the crashing.** Hey, it sucks to be on the frontier, software sometimes breaks and needs time to be refined. I just wish I could go more than 5 minutes without Intellisense crashing or more than 15 minutes without Xamarin Studio crashing. Coding F# with functioning Intellisense would be so groovy...

What's the good news? The [Xamarin Studio F# integration is open source](https://github.com/fsharp/fsharpbinding/tree/master/monodevelop)! That means I should "put up or shut up". I have a small amount of experience writing XS addins and a small amount of experience coding F# - I figure this qualifies me to at least send a pull request. Wish me luck!
