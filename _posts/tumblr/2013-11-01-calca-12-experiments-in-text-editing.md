---
layout: post
title:  "Calca 1.2 - Experiments in Text Editing"
date:   2013-11-01 16:34:21 GMT
redirect_from:
  - /post/65701122908
  - /post/65701122908/calca-12-experiments-in-text-editing
---



Let’s face it, text editing text on iOS isn’t the most enjoyable exercise. Text input is fine and dandy, I love flinging my thumbs or resting the iPad on my lap. But *editing*, no sir. That just sucks.

I’m talking about moving the cursor to make a correction, or moving it to change a block of text, or selecting a block of text. None of that is easy. Version after version of iOS has the same text editing features, so Apple doesn’t know how to make it any easier either.

In July, I released [Calca](http://calca.io) - a text editor based calculator that came from the desktop world. In the desktop world modifier keys, cursor keys, and crazy key mappings make editing fast. There are even whole religions devoted to [key](http://www.vim.org)[mappings](http://www.gnu.org/software/emacs/).

But we have none of that on iOS. On iOS, you get an uncontrollable keyboard and the ability to give it an ugly appendix. Calca takes advantage of this and makes a helpful appendix with common symbols and auto completion options.

![image]({{ "/assets/tumblr/65701122908_0.png" | absolute_url }})

The appendix (technically a “keyboard accessory”) does a lot to help input, does nothing for *editing*. You were still stuck with slow cursor movement and tricky block selection.

I wanted to make editing easier and ended up implementing three small features that, I think, accomplish that.

**Tapping a Number Selects It**

![image]({{ "/assets/tumblr/65701122908_1.png" | absolute_url }})

When you tap an arbitrary location while editing, the cursor will jump to the beginning or end of a nearby word. This is almost always what you want to happen.

But when it come to numbers, you often want to replace it outright. This means that you have to do some fancy backspacing or block selection. 

To make selection easier, Calca will select the whole number when it is tapped. This makes typing in new values very easy and will be useful for the third gesture.

**Swiping Left and Right Move the Cursor**

![image]({{ "/assets/tumblr/65701122908_2.png" | absolute_url }})

Fine cursor movement is normally achieved by tapping, holding, and waiting (forever) for the magnifying glass to appear.

To make precision cursor movement easier, Calca responds to horizontal swipes and pans. When you pan your finger, the cursor moves at about the same rate giving the impression that you’re nudging it along.

I got the idea for this while playing with [Editorial](http://omz-software.com/editorial/) by omz:software. Editorial has a similar feature but is limited to only working on the keyboard accessory and is a bit slow.

Calca allows the gesture in the editor and the keyboard accessory giving you plenty of room to control it. It was also sped up so that the cursor speed matches your finger speed. I found this necessary to making it feel like you are really in control.

**Swiping Left and Right to Modify Numbers**

![image]({{ "/assets/tumblr/65701122908_3.png" | absolute_url }})

Editing a number usually involves typing in a new value using the keyboard. Thanks to the number selection gesture, this is now very easy.

But typing presents a distraction when using Calca - part of the fun is watching calculations update when you change values. But if you're staring at the keyboard then you'll miss the action. 

Now, when you swipe left and right, the number is incremented and decremented. This free's your eyes to explore the rest if the document and is a great relief from pecking keys on the keyboard. 

This mode is only active when a number is selected so it works very well as a follow up to the first tap gesture to select numbers.

Thank you for reading! I hope you'll give [Calca 1.2 on iOS a try](https://itunes.apple.com/us/app/calca/id635757879?ls=1&mt=8)!
