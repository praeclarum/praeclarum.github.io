---
layout: post
title:  "Xamarin.Forms on the Web"
date:   2017-11-15 19:13:57 GMT
redirect_from:
  - /post/167525842703
  - /post/167525842703/xamarinforms-on-the-web
---



TLDR: I implemented a web backend for Xamarin.Forms so that it can run in any browser. It achieves this without javascript recompilation by turning the browser into a dumb terminal fully under the control of the server (through web sockets using a library I call [Ooui](https://github.com/praeclarum/Ooui)). This crazy model turns out to have a lot of advantages. [Try it here!](http://ooui.mecha.parts)


## A Need


I have been enjoying building small IoT devices lately. I've been building toys, actual household appliances, and other ridiculous things. Most of these devices don't have a screen built into them and I have found that the best UI for them is a self-hosted website. As long as the device can get itself on the network, I can interact with it with any browser.

There's just one problem...


## The Web Demands Sacrifice


The web is the best application distribution platform ever made. Anyone with an internet connection can use your app and you are welcome to monetize it however you want. Unfortunately, the price of using this platform is acuquiecense to "web programming". In "web programming", your code and data are split between the client that presents the UI in a browser and the server that stores and executes application data and logic. The server is a dumb data store while the client executes UI logic - only communicating with the server at very strategic points (because synchronization is hard yo). This means that you spend the majority of your time implementing ad-hoc and buggy synchronization systems between the two. This is complex but is only made more complex when the server decides to get in on the UI game by rendering templates - now your UI is split along with your data.

Getting this right certainly is possible but it takes a lot of work. You will write two apps - one server and one client. You will draw diagrams and think about data state flows. You will argue about default API parameters. You will struggle with the DOM and CSS because of their richness in both features and history. You will invent your own security token system and it will be hilarious. The web is great, but it demands sacrifices.

(And, oh yes, the server and client are usually written in different languages - you have that barrier to deal with too. The node.js crew saw all the challenges of writing a web app and decided that the language barrier was an unnecessary complication and removed that. Bravo.)


## Something Different


I was getting tired of writing HTML templates, CSS, REST APIs, and all that other "stuff" that goes into writing a web app. I just wanted to write an app - I didn't want to write all this boilerplate.

I decided that what I really wanted was a way to write web apps that was indistinguishable (from the programmer's perspective) from writing native UI apps. If I wanted a button, I would just `new` it up and add it to other UI elements. If I wanted to handle a click event, I wanted to be able to just subscribe to the event and move on. What I needed was a little magic - something to turn my simple app into the server/client split required by web apps.

That magic is a library I call [Ooui](https://github.com/praeclarum/Ooui). Ooui is a small .NET Standard 2.0 library that contains three interesting pieces of technology:

1. 

A shadow DOM that gives a .NET interfaces to the web DOM. It has all the usual suspects `<div>`, `<span>`, `<input>`, etc. along with a styling system that leverages all the power of CSS.
2. 

A state-synchronization system. This is where the magic happens. All shadow DOM elements record all the operations that have ever been performed on them. This includes changes to their state (setting their inner text for example) but also methods that have been called (for instance, drawing commands to `<canvas>`). This state can then be transmitted to the client at any time to fully mirror the server state on the client. With this system, all logic is executed on the server while the client renders the UI. Of course, it also allows for the client to transmit events back to the server so that click events and other DOM events can be handled. This is the part of Ooui that I am the most proud of.
3. 

A self-hosting web server (with web sockets) or ASP.NET Core action handlers to make running Ooui very easy. If I want to self-host a button, I simply write:

```csharp
var button = new Button { Text = "Click Me!" };
button.Clicked += (s, e) => button.Text = "Thanks!";

// Start a web server and serve the interactive button at /button
UI.Publish("/button", button);
```


I can do this from any platform that supports .NET Standard 2. I can run this on a Mac, Linux, Windows, Raspberry PI, etc.

Alternatively, you can host it on an ASP.NET MVC page if you want it up on the internet:

```csharp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        var button = new Button { Text = "Click Me!" };
        button.Clicked += (s, e) => button.Text = "Thanks!";

        // Return interactive elements using the new ElementResult
        return new ElementResult(button);
    }
}
```


Pretty neat huh?

But one more thing...


## Xamarin.Forms Support


The DOM is great and all, but what do .NET developers really love when you get right down to it? XAML. This little serialization-format-that-could has become the standard way to build .NET UIs. Whether you're writing a Windows, UWP, or mobile app, you expect there to be XAML support.

So I made XAML work on the web by implementing a new web platform for Xamarin.Forms. Now, any of your Xamarin.Forms apps can run on the web using ASP.NET.

Xamarin.Forms was not at all on my radar when I was building Ooui. Eventually though I realized that it was the perfect basis for a web version of Forms. I thought the idea to be a little silly to be honest - web developers love their CSS and I didn't think there was much point. But one day I heard someone ask for just that feature and I thought "now we're two".

I had never written a backend for Xamarin.Forms but found the process very straightforward and very easy given its open sourceness (e.g. I copied a lot of code from the iOS implementation :-)). There's still a bit of work to be done but Xamarin.Forms and Ooui are getting along like long-lost cousins.

Animations work, pages and layouts work, styling works (as far as I have implemented), and control renders are currently being implemented. Fonts of course are an annoyance and cause a little trouble right now, but it's nothing that can't be fixed.

Once I got Xamarin.Forms working on the web I realized how wrong I was for thinking this to be a silly technology. Writing web apps with the Forms API is a real pleasure that I hope you'll get to experience for yourself.

Now that I am officially releasing Ooui, I want to work on a roadmap. But for now I mostly just want to hear people's opinions. What do you think about all this? What are your concerns? How do you think you could use it? Do you think it's as cool as I do? (Sorry, that last one is a little leading...)
