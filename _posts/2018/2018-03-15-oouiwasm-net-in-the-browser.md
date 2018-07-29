---
layout: post
title:  "Ooui.Wasm - .NET in the browser"
date:   2018-03-15 15:20:33 GMT
redirect_from:
  - /post/171899388348
  - /post/171899388348/oouiwasm-net-in-the-browser
thumbnail: "/images/tumblr/171899388348_0.png"
---



I’ve been obsessing over my little [.NET web framework Ooui](https://github.com/praeclarum/Ooui) and am excited to announce that it is now capable of running fully in the browser as a web assembly. This means that Xamarin.Forms apps can run completely in the browser - without a smart server - giving .NET developers even more reach and options for distributing their apps. Try the [Xamarin.Forms XAML editor demo](https://s3.amazonaws.com/praeclarum.org/wasm/index.html) online or [read the getting started guide](https://github.com/praeclarum/Ooui/wiki/Xamarin.Forms-with-Web-Assembly) if you want to try it yourself.


### Demo


![image]({{ "/images/tumblr/171899388348_0.png" | absolute_url }})



This simple [XAML Editor running as a web assembly](https://s3.amazonaws.com/praeclarum.org/wasm/index.html) demonstrates the power of web assembly nicely. It demos Xamarin.Forms running in the browser and shows that all logic is working by enabling you to edit the displayed XAML.

I'm hosting it on S3 to drive home the point that this app is distributed as just a bunch of static files - all execution is done in the client browser. Once the app is loaded, you can turn off your network and everything will keep working. You can also inspect the HTML source to see that it's just a shell of an app.

You may be asking how this works. Web assembly is the latest browser tech to enable non-JavaScript languages to execute in the browser. In the past, the only way to create HTML5 web apps was to write JavaScript or to compile your app down to JavaScript (like [Netjs](https://github.com/praeclarum/Netjs) and [Fable](https://github.com/fable-compiler/Fable) do). This obviously works but has limitations because JS wasn't designed to be a low-level target language. Fortunately, the browser cabal that runs the internet has recognized this shortcoming and have created web assembly - a proper low-level target for programming languages to compile down to.

Web developers are no longer forced to work with JavaScript and thanks to the [amazing work of the mono team](https://github.com/mono/mono/tree/master/sdks/wasm), we can run full .NET code (.NET Standard 2.0) in the browser! This even works on mobile browsers. Wild!


### Easy as 1-2-3


You can now create a .NET web assembly app with just 3 commands:

```csharp
dotnet new console
dotnet add package Ooui.Wasm
dotnet build
```


This will create web assembly build of your app ready to be run in any modern browser. The build is tucked away in a **dist** subdirectory of your **bin** directory.

Now that app is quite boring and will just print “Hello World!” to the console instead of displaying the words. To fix this, we can edit the program:

```csharp
UI.Publish("/", new Span("Hello World!"));
```


That one line of code will add a span to the HTML document to display the greeting. Every Ooui.Wasm app declares its initial UI by publishing that root element. Of course, the app is free to change things afterward. Check out the [getting started guide](https://github.com/praeclarum/Ooui/wiki/Web-DOM-with-Web-Assembly) for a longer example.


### Xamarin.Forms in Web Assembly




Since Ooui.Forms already implements a backend for Xamarin.Forms, Xamarin.Forms works out of the box in web assembly!




I wrote a [getting started guide for Xamarin.Forms](https://github.com/praeclarum/Ooui/wiki/Xamarin.Forms-with-Web-Assembly) that walks you through a complete example.


### Pros and Cons of Web Assembly


Let’s compare writing an app with web assembly to writing a more traditional web app with Ooui.

**Traditional**

* **Pro:** Uses billion-year-old HTML that works everywhere in the known universe. Google can read it, bots can read it, and it probably might work in Internet Explorer.
* **Con:** You need to run a web server that can execute code and that server will have to scale as your users increase.
* **Pro/Con:** Data is shared by default between users (since it’s all coming from the server)

**Web Assembly**

* **Pro:** No smart server needed - you can host your app on a static web server such as Amazon S3, Azure Blob, another CDN, or a low-power device.
* **Pro:** Apps can be cached to be able to run in disconnected scenarios.
* **Con:** Relies on fancy new support in modern browsers. Fortunately support is ubiquitous today, but this won’t be working on Windows XP.
* **Pro/Con:** Data is private by default between users (since everything is running locally)

The last items were marked pro/con because it really depends on your app whether data between users should be easily shared. If you’re making a social network, then yes you want easy sharing. If you’re making a private journaling app, maybe local is right.


### Comparison with Blazor


[Blazor](https://blogs.msdn.microsoft.com/webdev/2018/02/06/blazor-experimental-project/) is an amazing <strike>product</strike> experiment from Microsoft that also uses mono's web assembly support. Instead of exposing the DOM and classes and objects as Ooui does, it uses Razor templates composed through "components" to build your UI.

You can think of Ooui.Wasm and Blazor as application frameworks running atop a common runtime (mono wasm). Ooui was designed to feel like classical native object oriented UI development which enables it to run even higher-level app frameworks like Xamarin.Forms. Blazor was designed to feel like modern React-style web development where you're writing HTML templates. Pick your poison.


### Big Thanks


It’s surprising how little of Ooui had to change in order to work as a web assembly. This is thanks to the amazing effort of the mono team. I want to especially thank [Rodrigo Kumpera](https://github.com/kumpera) for helping me get everything working.
