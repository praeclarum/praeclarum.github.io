---
layout: post
title:  "The Technology of iCircuit 3D"
thumbnail: "/images/2021/icircuit3d_thumb.jpg"
tags: announcement
---

Today I’m pleased to announce the [macOS release of iCircuit 3D](https://apps.apple.com/us/app/icircuit-3d/id1539977373)! Last week I released the iOS version to a wonderful reception and I’m happy to now be able to give all the Mac users out there the same experience. iCircuit 3D has been a work of passion over the last couple years and I thought I would take a moment to describe some of the more interesting technical aspects of it.

So here is a very extended colophon for the app. Maybe you can get some ideas for your next app from it!

<a href="/images/2021/icircuit3d.jpg"><img src="/images/2021/icircuit3d.jpg" alt="iCircuit 3D showing switches and LEDs" /></a>

## Programming Language and Style

iCircuit 3D is written 100% in C# using Xamarin.iOS and Xamarin.Mac. I started with C# 7 but C# 9 had already been released by the time I shipped the app. :-) The null reference checking feature of C# 8 has been especially useful in tracking down bugs.

The app is very object oriented. In fact, this may be the largest OOP app I’ve ever built. This is funny considering that I am quite an advocate for functional programming and more reactive style UI design these days. But react style programming of a real-time engine like this is not trivial and I found myself trailblazing a bit more than I like. Like it or hate it, OOP is well understood at this point.

I decided to go OOP, in some ways, to rid myself of writer’s block too. I’ve been writing OOP programs since the 1990s and I am very comfortable with the abstraction and patterns. Sometimes when you’re stuck, the best thing to do is to lean into your strengths.

### Properties FTW

A mental road block I often run into when starting a new app is how to handle serialization and undo buffers. I decided to solve both problems in a generic way that would require very little on-going work. The key was to base both undo and serialization on properties of objects. 

Serialization is a simple process thanks to advanced libraries like [Newtonsoft.Json](https://www.newtonsoft.com/json). I'm able to serialize and restore entire object graphs (thanks to its object reference handling) and only have to be careful that I don't serialize more than I need to. I also chose to use the BSON serialization format so that the app could handle binary data more efficiently. (Circuits can contain binary data thanks to image and board import facilities.)

The undo (and redo) system of the app is based on property changes. When a property changes, an undo change event is registered with the OS (Apple has `NSUndoManager`). That event captures the property's value before and after the change. When the user undos and redos, the system just has to figure out which of those values to restore to the property.

All of this is achieved with property definitions such as this:

```csharp
public class Resistor : CircuitElement {
    double resistance = 1000;
    [Inspector (Units = "Ω", Min = 0, Max = 1e12)]
    public double Resistance {
        get => resistance;
        set => SetUndoableProperty (ref resistance, value, () => {
            InvalidateMaterial ();
            InvalidateAnalysis ();
        });
    }
}
```

The trick is to call `SetUndoableProperty` which is very similar to the MVVM helpers standard `SetProperty`. It checks if the value actually did change, and, if so, calls an inline change handler (this example invalidates some things) and triggers the `PropertyChanged` event. `SetUndoableProperty` takes just one extra step and registers that change with an undo manager. I found that you want to be explicit about what gets registered with the undo manager because not all properties are set by users and don't need to be registered. That's all that's needed for me to get serialization, undo support, and inspector UI support (thanks to reflection). 

If you adopt the pattern of default object constructors and properties that can change any time, you can get these cross-cutting features essentially for free. I love how much power can be put into a single property.

I've talked about this a bit more on Merge Conflict over the last couple years. (Gosh I've been working on this app for awhile.) Give them a listen if you're interested in details!

<iframe src="https://player.fireside.fm/v2/UDzB5o3V+Hg_UIk4b?theme=dark" width="740" height="200" frameborder="0" scrolling="no"></iframe>

<iframe src="https://player.fireside.fm/v2/UDzB5o3V+RX7Y9rNO?theme=dark" width="740" height="200" frameborder="0" scrolling="no"></iframe>

### Lots of Multithreading

My biggest gripe with OOP is its inability to handle shared-memory multi threading. And, honestly, the style basically guarantees your code will have race conditions. In iCircuit 3D there are 4 kind of threads that all have to interact with each other:

1. The main UI thread is responsible for drawing the 2D UI and handling user interactions
2. The 3D render thread is responsible for drawing the environment (one for each view)
3. The electronics simulation thread, well, simulates things
4. The background threads that perform the operations that are too slow to perform on the other 3 threads

In other words, there is a lot to synchronize in a mutation heavy environment.

I ended up using .NET’s monitor-based locking primitive `lock (thing) { }` to do fine grained locks over critical data structures. From my past experience writing multithreading-heavy apps, I knew that the lock primitive was very efficient. Not safe, but efficient. ;-)

Safety came from designing thread-safe interfaces to objects and being very strict with myself in how callbacks are executed. The nemesis of locking is deadlocks. Deadlocks are usually caused by holding a lock while executing unknown code (from a callback, an enumerable, or an event). I sacrificed a little performance for stability by ensuring I only held locks while executing code that I was sure was callback and recursion free. It's hard sometimes but the benefits of having thread safe code are innumerable. Also, the good news is that deadlocks are actually easier to debug than race conditions. If the app deadlocks, pause it, find what every thread is blocking on, slap yourself on the wrist, and fix the code.

I’m sure there are race conditions galore in the app, but multithreading bugs just have not been an issue... usually... But it takes diligence.

### Apple Only

While C# and .NET certainly lend themselves to cross-platform code, I decided to focus primarily on Apple APIs.

In order to make progress on the app, I abandoned cross-platform support and focused on leveraging the magnificent Apple APIs. This was a great decision. Before I made this decision, I would have to consider how to do fancy things on 3 different platforms. Easy stuff is easy, but hard stuff (like 3D renderers) are, well, hard. Focusing on one platform made the app better because I could optimize that experience without spreading my time out across platforms.

## The 3D Renderer

The main user interface is a 3D environment powered by Apple’s SceneKit technology. SceneKit is basically a game engine built into every version of iOS and macOS. I love it because it is both feature rich and performant. I’m continually impressed by it being able to handle whatever I throw at it (within reason of course) and continually getting graphics improvements.

Some features I take heavy advantage of include:

### Thread Safety

Apple was kind enough to make SceneKit thread safe. This means I can usually access it from whatever thread I want. I did notice, however, that there are some race conditions if you don't execute your code in SceneKit's transactions. It's not hard, you just have to make sure to write:

```csharp
SCNTransaction.Begin ();
try {
    // Code that does SceneKit things
}
finally {
    SCNTransaction.Commit ();
}
```

whenever you want to manipulate rendered objects. With that pattern I could confidently manipulate the scene graph from whatever thread I wanted.

### Physically Based Rendering

<a href="/images/2021/icircuit3d_battery.jpg"><img src="/images/2021/icircuit3d_battery.jpg" alt="iCircuit 3D rendering of a battery" /></a>

PBR is the new hotness if real-time realistic-looking rendering. It’s actually a technology developed for movies to make visuals look less, well, plasticky and now I’m able to run it on my phone. Amazing.

PBR requires that you provide three textures for every object: its color, its roughness, and its metalness. You can imagine what color does. Roughness lets you control how rough or shiny an object is and metalness controls the color of light reflected off the object. The fact that you can vary these levels on a pixel-per-pixel level enables a lot of nice effects. This is most notable on AA batteries who have a diffuse body, a shiny metallic stripe, then dull metallic contacts. That’s all done using a real-time generated texture.

### LED Lights

Once you have a good lighting model, you need some good lights. I decided to make every LED a light source. This actually overwhelms SceneKit because it has a maximum of 8 light sources, but things are pretty fun up until that point. This is a short section, I just thought the LED lights were fun. :-)

### Skeletal Meshing for Wires

<a href="/images/2021/icircuit3d_wires.jpg"><img src="/images/2021/icircuit3d_wires.jpg" alt="Triangles of a wire" /></a>

One fun feature of iCircuit 3D is that its wires arc around the environment like real wires. I am using skeletal meshing, a technology that is usually used for character movement in games, to accomplish this.

I use a 3D Bézier curve that starts by matching its direction to the port it's attached to. It then travels in a direction towards the other object and then finally curves again to meet that objects' port. It's a lot of math to figure out (shout out to [Calca](http://calca.io) for helping me) but really adds some pizzaz to the app. I move skeletal control points to lie on that curve and then allow SceneKit's code to move the geometry of the wire to match that skeleton.



### Environment Maps

<a href="/images/2021/icircuit3d_env.jpg"><img src="/images/2021/icircuit3d_env.jpg" alt="A reflection of the environment" /></a>

PBR only looks good if you give it a nice environment. A friend and I spent an evening in a craftsman’s workshop taking photography to light this app using a cheap 360 degree camera. Then Wilderness Labs’ Bryan Costinach let me take photos of his lab to add some variety to the app. To add a bit more user control I also added SceneKit’s sky generator that tracks the current time of day, and users can select their own colors. 

### Custom Shaders

SceneKit has some beautiful light rendering, but sometimes you want to take control. The API makes this easy by being able to inject little shader snippets into each model by assigning a string to a material property. As someone who has been writing and using 3D renderers their whole career, this is an absolute delight. I love the freedom to just code up any graphics I want without mucking around with compiling those programs.

### Dynamic Geometry

<a href="/images/2021/icircuit3d_cap.jpg"><img src="/images/2021/icircuit3d_cap.jpg" alt="Triangles of a capacitor" /></a>

This isn’t a feature specific to SceneKit, but one that I take great advantage of in order to make parts parametric. I have some high-level modeling abstractions in code that allow me to use virtual lathes, virtual mills, and constructive solid geometry. This allows me to define the geometry of parts in code and change them as I desire. Think OpenSCAD or OpenJSCAD but better. ;-)

This is notable in electronics like the capacitor whose shape changes depending on capacity of the device. It also allows one to control the radius of the wheel independent of the width for instance.

### Geometric Instancing

Like any good engine, SceneKit can efficiently render multiple instances of the same geometry in one frame, and I take advantage of this. My code does its best to share as much geometry between part instances as possible while still varying material properties. This is achieved by a per-part geometry cache that uses part parameters as keys. It was tricky to setup but is worth it to have very efficient geometry loads and a high-level overview of how much I have allocated.

## Physics Simulation

The last major feature of SceneKit that I use is its built-in physics engine. This engine has all the standard support for collision detection, gravity, and joints. iCircuit 3D makes use of all of these.

When I began the app, I focused on the interaction of picking up a part and moving it around on the work surface. I wanted to nail that touch interaction on the iPad. My thought was that if I make it fun and easy to move parts around, people will be encouraged to try building larger and larger circuits. I found that gravity was my friend. It made picking up parts and setting them down feel more real, more natural than in traditional CAD apps.

Once I had gravity, I had to have collision detection - two parts should never occupy the same space. This gives a sense of "thisness" to the parts, again, making them feel more real.

The last step was to integrate physical joints into the app. Joints are used in physics engines to glue two things together. It can either be a rigid joint (like glue) or a flexible joint like a hinge.

I added rigid joints to the app so you could build structures and panels. There are some physical primitives that can be made "sticky" and when another object is dragged over them, they become rigidly connected. I use these rigid joints for both breadboards and for building blocks. It's pretty interesting what you can build with it.

I do not allow users to create flexible freely, but I have two elements, a DC motor and a servo motor that create implicit physical joints. Thanks to SceneKit, I can control the velocity of the joints in order to simulate those motors. It's tricky code, and it's admittedly a bit buggy, but it really open up what the app can do.

## Electronics Simulation

iCircuit 3D uses the same robust electronics simulation engine as iCircuit. This is a pretty standard nodal electronics simulator with some notable features:

* It uses connectivity to only simulate parts that are wired to each other. This means you do not pay the cost of simulating extra parts in the environment. This is what enables you to liberally use Duplicate and Copy and Paste without having to worry about affecting the simulation engine.

* It can simulate Arduinos using my custom C compiler and interpreter called, surprisingly, [CLanguage](https://github.com/praeclarum/CLanguage).

I am able to share the simulation engine between the original iCircuit and iCircuit 3D. This is great for me because any improvements I make to the original will be automatically imported into this engine. As a solo indie developer, that's a lot less maintenance. And it also means all users of iCircuit with benefit from improvements to both apps.

This code sharing also allowed me to import the entire iCircuit library of parts into iCircuit 3D. That has really increased its usefulness.

## The End

I hope you enjoyed this tour of iCircuit 3D technology. If you found this interesting and think the app could interest you, I hope you'll [give iCircuit 3D a try](https://apps.apple.com/us/app/icircuit-3d/id1539977373)!

