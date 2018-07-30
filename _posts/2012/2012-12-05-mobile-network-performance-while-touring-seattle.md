---
layout: post
title:  "Mobile Network Performance while Touring Seattle"
date:   2012-12-05 19:15:00 GMT
redirect_from:
  - /post/37273215875
  - /post/37273215875/mobile-network-performance-while-touring-seattle
thumbnail: "/images/tumblr/37273215875_0.png"
---


I recently journeyed around Seattle to get a sense of the impact motion has on the network performance of mobile applications.

![Request and response times touring Seattle]({{ "/images/tumblr/37273215875_0.png" | absolute_url }})

The image above is a graph of how long it took to download the homepage of Google News (588 KB) while I was traveling by bus and train throughout the city. Blue bars are successful downloads and red bars (and time entries with no bars at all) represent errors. The height of the bar shows how long it took to find out if it was successful download or an error.

I wrote the [test app](https://gist.github.com/4210793) using [MonoTouch](http://xamarin.com) and ran it on an iPhone 5 with Verizon LTE.


## Observations



### When you're standing still, performance is pretty consistent and error free


Whenever I gave the phone time enough to pair with a cell tower, the performance was pretty consistent with 2.5 second response times with 588 KB of data (240 KB/s). I never received errors (go TCP!).


### You will get spikes errors when moving


While most downloads took 2.5s, I received many 5s spikes. This is **100% increase** in time. There was a even a time (look in the last 1/3rd of the chart) where 5s became the average and I received 10s spikes.

The spikes happened quite spuriously so I can only guess that they occured in cell tower transition zones.

While TCP is an insanely resiliant protocol, it would still fail from time to time. In a 2 hour interval, I recorded 2 errors that weren't tunnel-related. This means that there was a **1% error rate** while moving. My only guess is that these happened whenever I was switching towers and the request happened just then.


### 3G sucks compared to LTE


Do you see those blue spikes near the center just after a little gap (the Mount Baker tunnel)? The one that peaks at more than 50s? That's 3G my friends: slower downloads by an order of magnitude and a huge variance increase. I can only wonder what this graph would be like if I was stuck on 3G the whole time...


### Timeout and ReadWriteTimeout control these spikes


For the test, I used a [WebRequest.Timeout](http://msdn.microsoft.com/en-us/library/system.net.httpwebrequest.timeout.aspx) of 20 s and a [ReadWriteTimeout](http://msdn.microsoft.com/en-us/library/system.net.httpwebrequest.readwritetimeout.aspx) of 20s.

You can see the affect of Timeout with all the errors that cap at 20s. Mono gave a "System.Net.WebException: The request timed out" at that mark. This is great because it's very predictable.

So, how do bars go above 20s? That's the effect of ReadWriteTimeout. This is the timeout that affects each Read and Write call to the response object. I was using a 16 KB buffer for my reads so this means that I did about 37 read operations. Since each of these had a timeout of 20s I could have ended up waiting up to 740s (12 minutes)! Thankfully that never happened.

About 50% of the time, ReadWriteTimeout was able to finish the download. On the other 50%, I ended up receiving "System.Net.WebException: The operation has timed out." exceptions. I'm not sure if I see much value in setting it up - I would prefer that errors come sooner rather than later.

The system default of 300s (5 minutes) seems beyond ridiculous to me. Don't use that value.

TL;DR ReadWriteTimeout is an important value that you should think about.


### Thank God for ConnectFailures


While travelling through a tunnel, the request would fail immediately with "System.Net.WebException: Error: ConnectFailure (No route to host)".

These errors are represented by the white gaps in the chart. In the face of those 38 s errors, this is wonderful.


### All errors were represented by WebExceptions


I've always wondered if a try-catch block of:

```csharp
try {
    // Do some System.Net.WebRequest stuff
} catch (System.Net.WebException) {
    // OMG. OMG. OMG.
}
```


one that caught only [WebException](http://msdn.microsoft.com/en-us/library/system.net.webexception.aspx) - was sufficient to catch all network related errors when using [WebRequest](http://msdn.microsoft.com/en-us/library/system.net.webrequest.aspx). It turns out that yes, yes it is. One less thing to think about...

Here are *all* the errors MonoTouch emitted during this test (in Type.Name + Exception.Message format):

* System.Net.WebException: Error: ConnectFailure (No route to host)
* System.Net.WebException: The request timed out
* System.Net.WebException: The operation has timed out.


## Code Setup


* **Request Class:**`System.Net.HttpWebRequest`
* **URL:**[http://new.google.com](http://news.google.com)
* **[Timeout](http://msdn.microsoft.com/en-us/library/system.net.httpwebrequest.timeout.aspx):** 20 s
* **[ReadWriteTimeout](http://msdn.microsoft.com/en-us/library/system.net.httpwebrequest.readwritetimeout.aspx):** 20 s
* **HTTP Keep-Alive:** No
* **ReadBuffer:** 16 KB
* **Request Interval:** 30 s

The actual code for the netowork request can be found in the `HandleTimer` method in the [gist for the test app](https://gist.github.com/4210793#L98).

The app disabled the idle timer of the phone so that it could run continuously in the foreground. Every 30 seconds, it would download the full 588 KB of Google News. In all, I figure I burnt through 8% of my monthly data allowance.


## Test Procedure


* **Device:** iPhone 5 (white)
* **Carrier:** Verizon (LTE, usually)

I started the test by getting a slice of pizza at [Pagliacci](https://maps.google.com/maps?q=4529+University+Way+NE+Seattle,+WA+98105&hl=en&sll=47.775216,-122.310617&sspn=0.007153,0.010343&hnear=4529+University+Way+NE,+Seattle,+Washington+98105&t=m&z=17)'s in the University District of Seattle. I figured this would be a good way to establish a baseline for staying still, and I was very hungry. I then took a bus to [Westlake](http://www.soundtransit.org/Rider-Guide/Westlake-Station.xml), downtown, wherein we entered the transit tunnels and I transferred to the light rail. I took the light rail through the tunnel and back into the open air as far as [Columbia City](http://www.soundtransit.org/Rider-Guide/Columbia-City-Station.xml). This journey consumed the first hour of the test. There, I switched trains to head back into the city. The trip back was essentially the same and even included another stop for pizza in the name of consistency (at [Sbarro](https://maps.google.com/maps?q=400+PINE+ST.+%23332,SEATTLE,WA,98101) this time).
