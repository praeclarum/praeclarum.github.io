---
layout: post
title:  "Introducing the iCircuit Gallery"
date:   2015-06-03 18:14:47 GMT
redirect_from:
  - /post/120625815178
  - /post/120625815178/introducing-the-icircuit-gallery
thumbnail: "/images/tumblr/120625815178_0.png"
---



**TLDR;** I wrote a [website to share circuits made with my app iCircuit](http://gallery.icircuitapp.com) and I hope you’ll check it out.

![image]({{ "/images/tumblr/120625815178_0.png" | absolute_url }})


## Finally, a place to share


iCircuit users create amazing things. For the past 5 years of reading support emails, I have been privy to just a fraction of these wonders. Circuits far bigger than I ever thought iCircuit could handle - circuits that were clever and required me going back to my college texts to understand - and circuits that just made me laugh. I learned something from each of them.

It was a shame that all these wonders were hidden in my inbox. Well, **no more**.

Introducing, [the iCircuit Gallery - a community driven web site full of circuits](http://gallery.icircuitapp.com).

Now iCircuit users have a place to upload their and share circuits with the world. Each circuit is lovingly rendered in SVG and can contain rich textual descriptions of the circuit. Even if you’re not an iCircuit user, you can still learn a lot from the gallery.

I have seeded the site with the standard example circuits and Windows Phone users have (believe it or not) been able to upload circuits for years - so the site has some initial work in it already. But,

**I am asking iCircuit users to share their designs** - big or small - novel or standard - brilliant or otherwise. Share them with the world! There is great satisfaction to be had in sharing your work with others. I hope also to see educational examples pop up that take full advantage of the ability to document the circuit.

Simply click the Upload button, create an account (email optional), and pick the files off your device. Right now, that means Mac and Windows users have the easiest time with the gallery. I am working on iOS and Android updates to make uploading a snap there too.

I am very excited to see your designs!


## Future Improvements


I have lots of ideas on how to improve upon this initial release but hope to get some feedback from the community before pursuing any of them. For example, I hope to add Tags to help organize things and Comments if contributors desire.

Also, I will be integrating the gallery into the app to make browsing and uploading easier. Keep your eye out for updates!


## Colophon


Oh my, I wrote a website! With servers and all that. Part of the reason it took me 5 years to write this thing is that I am scared to death of running servers. My ability to manage a server only gives it a life span of a few months before some hacker is using it as a spam bot.

So what’s changed? App hosting is what’s changed. I adored Google App Engine for it remedied the whole server problem - host apps instead of servers - genius! They provided a great database and a great toolset.

But it wasn’t .NET and I always wanted to run the iCircuit engine on the server.

And then Azure came along. Azure has a million enterprisy “solutions” and one awesome service called *Mobile Services*. But they their *Cloud Service* was the most confusing thing ever. It acted like an app host but also acted like a server. Which was it? So very confusing.

Well, Azure fixed that with a *Web Apps* service. Finally, after that little marketing spin and an assurance that I’m not managing a server, I became a customer.

Building the site was a snap with ASP.NET MVC. My only possible mistake is that I’m using Azure’s Table Storage - not sure how that decision will pan out. I foresee a future of migrating to SQL...

I am also scared to death about cloud pricing. Every page on the site has an HTTP and memory cache of 5 minutes. It’s ridiculously high. Almost as ridiculously high as my fear of cloud service pricing.

But there’s only one way to find out...
