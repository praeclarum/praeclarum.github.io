---
layout: post
title:  "Drone Builder - A Story of Drones, React, and F#"
date:   2015-09-14 20:22:32 GMT
redirect_from:
  - /post/129094397488
  - /post/129094397488/drone-builder-a-story-of-drones-react-and-f
---



[![image]({{ "/assets/tumblr/129094397488_0.png" | absolute_url }})](http://drone.mecha.parts)

[Drone Builder](http://drone.mecha.parts) is a site I created to
play with different DIY drone (multicopter) designs.

Building a drone isn't rocket science but there is a lot to learn when making
your first one. You first have to learn what parts you need and what all their
parameters mean. Then you have to learn how they combine to
produce different effects. On top of it all, you have to do it on a budget.

It's a lot to take in, but it's also a trying task when you know all of that.
You still have to track down shipment times, compare reviews, maintain Excel
sheets - it's a messy process.

So, [Drone Builder](http://drone.mecha.parts).

The UI is split into two areas: designs on the left and components on the right.
Each component has a list of products sold by online merchants
(Amazon and Banggood).

As you choose products on the right side, a design is built up on the left side.
If you choose multiple products for a component, multiple designs will be built
with all the possible combinations.

That combination of designs is the true power of *Drone Builder* - not only can
you design one drone, but you can easily design multiple variations and
compare them.

It's a fun little app, I hope you'll [give it a try](http://drone.mecha.parts)!

But you're not here for the drones, you want to know about this F# and React
thing.


## Historically...


To explain why I like React, let me compare it to the traditional way GUI apps
are built.

I started building UIs with Visual Basic. In those days, application logic and
UI logic mutated a large UI tree to create user experiences.

Well, it's still how we do it. The HTML DOM is a large tree that we can
manipulate with JavaScript. Building apps in HTML is roughly how we did it in
VB. We may use fancy binding libraries nowadays, but we're still mutating some
application data and then mutating a UI tree to match it (and the other way
around).

But, but, but. Time marches on and ideas evolve. We started to see some flaws
with this architecture for apps.

First, it makes parallelism hard - if objects
are mutated anytime, by anyone, then it's hard to write parallel tasks that
you can trust.

Next, we started to notice the dependency graphs were becoming incomprehensible.
If a tap mutates a property of object A resulting in an event that mutates
a property of object B that then mutates a property of A - we get ourselves
into a potentially endless update cycle. We've all added code of the like:

```csharp
void HandleEvent() {
    updatingUI = true;
    UpdateUI();
    updatingUI = false;
}
```


All to break the mutation dependency chain for a brief moment. (Usually to
guard against over-zealous UI events firing.)

Even if you manage to avoid cycles, you have a wild graph of objects with
a plethora of implicit references - both explicit references and implicit
references from events with closures. That is to say, you are creating fertile
ground for memory cycles that keep objects around past their welcome.

To combat this, one usually has to write "unbinding" code. This takes the form
of unsubscribing from events and disposing of objects we know to be useless.

It feels a lot like writing destructors in C++ - simple enough to explain: for
every event you subscribe to, make sure you unsubscribe. It's a bookkeeping
exercise; but who likes to keep books? One missed unsubscribe and you have
a dangling object eating your memory and resources.

Lastly, mutation and its destruction of data becomes undesirable. Building an
undo buffer becomes tricky business if we routinely overwrite data. Rolling back
to a valid state after a failed operation is very tricky business. But these
are trivial problems to solve when you don't destroy data.

The enemy has been identified as mutation - both mutation of application data
and mutation of the UI tree.


## React for Clean UI Layers


**React enables creating UIs without mutation and rewards you
for not mutating your model.**

React flips this model by treating the UI like any other data structure.
HTML entities, such as `div` - the analog of native "views", become
light-weight objects instead of large and complex OS resources.

The idea is to [map](https://en.wikipedia.org/wiki/Map_(higher-order_function))
your application state into a light-weight UI tree. Data mapping is a familiar operation
to any functional programmer and any .NET programmer that loves
[LINQ](https://en.wikipedia.org/wiki/Language_Integrated_Query).

We never mutate the DOM directly. Instead we just keep **creating** new UIs -
never destroying with mutation.

React then takes on the
onerous task of synchronizing that tree with the heavy DOM. This is all
done implicitly on behalf of the programmer.

Generally speaking this is a
heavy-duty process, but its performance can be drastically increased
if you use immutable data. This is because React can [cache the results of
previous generations](http://facebook.github.io/react/docs/advanced-performance.html)
if it is told that data hasn't changed. The only way
to know if data hasn't changed is to compare it to old data - something that
can only be done if you don't destroy the old data. Thus, immutability.

Writing these `map` functions can get a bit tedious
- especially when designing UIs - so React
introduces "components" with the
[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html) syntax.
Each of these components maps
a bit of your application state to UI state using declarative
HTML syntax. Instead of `map`` functions, you write markup *templates*.
For those familiar with XAML, this is very analogous to a XAML
page binding to a view model.

React is nicely architected with an
emphasis on composing apps from many of these small components - each
responsible for just a small part of the UI. When you combine all these
little components, you can build up an information-rich page.


### Updating Application State


There are events in React, but you don't handle them the way you did in VB.
That is, you don't mutate the app state, then mutate the UI component handling
the event.

Instead of mutation, you **clone** the entire application state while
making precision substitutions
in that clone. This clone preserves the old application state while also
giving the illusion of mutation.

You then notify the **root** of the UI tree - a component - that the app state
has changed. It re-maps itself (a process called "rendering" in React) and
recreates the UI tree. The DOM is subsequently (implicitly) mutated to match
that tree.

You end up with an app that centralizes app state changes. Facebook has even
gone so far as to codify such centralization in their
[Flux](https://facebook.github.io/flux/docs/overview.html) library.


## About that Persistence...


To ameliorate the cost of cloning, persistent
data structures (or, immutable data structures) are used. These are designed
to make this "clone with substitutions" trick easy on the CPU and
memory.

There's just one problem in this new React world - JavaScript.

I have nothing against JavaScript - I find it to be a rather enchanting
language in fact - but it was not designed with immutable data structures in
mind. It has no syntax to help declare them. It has no syntax to clone them.
It only knows about reference equality - not structural.

Facebook, the creators of React, recognized this and built another library
to help out. This one is called [Immutable](https://facebook.github.io/immutable-js/).
It's a brilliant little library (50KB minified) that adds a lot of persistent
data structures to JavaScript. If you're willing to forego JavaScript standard
way to create objects, then this library puts you well on the way to success.

But, but, but. *Immutable* is great, but there's a bit more needed to write
persistent transformations than what a set of generic data structures can
provide.

Ideally, you will have a programming language that takes immutability seriously.
Something like F# (or Elm, or Swift, or ...).


## F# to the Rescue


Not only does F# have immutability baked into its design, but it has a large
mature library of algorithms, data structures, and abstractions to help you
write the logic for your app.

When I think
of F#, I think of the `Seq` type. This is your generic pull-based infinite
stream of data and F# has a wonderfully powerful set of operations for working
with them in non destructive ways. It's a very useful tool to have at your
disposal, and it's a missing feature of JavaScript.

For data modeling, F# also has **union types** and **record types** both with
automatic structural comparison and hashing. These types are more specific
than "plain old objects" and can be used to create more precise models of
your problem.

From a programming standpoint, F# is great due to its simple and powerful
syntax. Functions are quick to define and easy to combine into chains. The
syntax is driven by whitespace so its easier to refactor and move code blocks
around in than, say, our curly-brace endowed languages.

And let's not forget F#'s other advantage: F# Interactive. It is a REPL
that allows you to execute code while you're writing it. F#'s take on the
REPL - F# Interactive - has nice IDE integration that makes writing apps
an amazingly satisfying experience.

If you would like to read more from me about using F# to create GUIs you
can look at [my slides from .NET FRINGE 2015](http://www.slideshare.net/frankakrueger/functional-guis).

But why am I talking about F#, isn't Drone Builder a web app?


## FunScript is Amazing


There is an insane library out there called [FunScript](http://funscript.info)
that can output
JavaScript code from your F# code.

Why do I say "insane library" and not "cool transpiler"? That's because of its
implementation. It turns out that F# has some amazingly powerful reflection
capabilities that include the ability to retrieve the abstract syntax tree (AST)
of your entire app.

Constructing the AST is the first step to building a compiler or transpiler.
Normally you write a parser, and then a type system, and then a module system...
You then write tricky code to add types to expressions and create data
structures to form the AST.
It's a lot of work. But it's exactly the work the F# compiler already performs
whenever you compile your app. The genius of F# is that it makes the results
of that effort (the typed AST) available to you at runtime.

All you have to do is mark the modules of your app with the
[ReflectedDefinition](https://msdn.microsoft.com/en-us/library/Ee353643.aspx)
attribute. With that, the F# compiler will retain the AST and make it available
to your app (and libraries like FunScript).

FunScript, armed with the full F# AST, is then able to generate JavaScript. This
process, in general, is difficult and error prone (translating between two virtual
machines) and FunScript handles it with aplomb.

It has an amazingly simple way
to replace F# expressions with JavaScript versions using just an attribute.
This little trick enabled the FunScript authors to port large swaths of the
F# standard library (Core) to JavaScript and also makes it easy for your
app to interact with other JavaScript libraries and the DOM not covered
out of the box.

One other great bonus for using F# with FunScript is IntelliSense. Dynamically
typed languages like JavaScript are hard to provide completion info for.
But for statically typed languages, like F#, code completion is nigh trivial.
That is to say, I get full IntelliSense as I'm coding my web app.

Part of that wonderful editing experience is thanks to the TypeScript team
and their effort towards wrangling JavaScript libraries to publish
"declaration" files. These files add type information to otherwise untyped
JavaScript libraries. FunScript is able to use those TypeScript declaration files to provide
IntelliSense for working with external JavaScript libraries and the DOM
itself. It's fantastic.


## Putting it all Together


So how do you build one of these React + F# apps? Let me walk you through
Drone Builder's architecture.


## The Data Model


Let's with the data model. The usual product-based suspects are
declared:

```csharp
type Component =
    | Frame of FrameOptions
    | Motor of MotorOptions
    | Esc of EscOptions
    | Propeller of PropellerOptions
    | FlightController of FlightControllerOptions
    | PowerDistribution
    | Battery of BatteryOptions
    | RadioReceiver of RadioReceiverOptions
    | RadioTransmitter of RadioTransmitterOptions

type Product =
    {
        Name : string
        Key : ProductKey
        Url : string
        ImageUrl : string
        DeliveryTime : int
        Price : float
        Currency : string
        Components : (int * Component)[]
    }

type DesignComponent =
    {
        Key : string
        ComponentInfo : ComponentInfo
        Component : Component
        Product : Product
    }

type Design =
    {
        Key : string
        Components : DesignComponent[]
        Purchases : (int * Product)[]
    }
```


That's it. These types - 3 records and 1 union - comprise most of the data
model. `Products` represent something that you can purchase online and
contain a set of `Components` (and quantities). There is not a 1-1 mapping
between products and components because online merchants love to bundle
things together.

A `Design` and `DesignComponent` is one specific way to build a drone. They are
calculated from an `analyze` function. More on that later...

There are also the `Options` types - these are just additional bags of data
attached to each component class. Here's the `MotorOption` to give you a flavor:

```csharp
type MotorOptions =
    {
        Weight : float
        VelocityConstant : float
        Diameter : float
        MaxCells : int
        Model : MotorModel
    }
```


(Note that I'm able to make use of F#'s units of measure.)

Products are assembled together into a big global variable called `products`:

```csharp
let products : Product[] =
    [|
        {
            Name = "EMAX MT2204 KV2300 + ARRIS 12A 2-3S ESC"
            Key = "A-B00Y0J5WLY"
            Url = "http://www.amazon.com/dp/B00Y0J5WLY/?tag=mecpar-20"
            ImageUrl = "http://ecx.images-amazon.com/images/I/51bguFHIyFL.jpg"
            DeliveryTime = 6*7
            Price = 105.00
            Currency = "USD"
            Components =
                [|
                    4, Motor { Diameter = 27.9; Weight = 25.0; VelocityConstant = 2300.0; MaxCells = 3; Model = MotorModels.M2204_2300 }
                    4, Esc { Weight = 12.0; ContinuousCurrent = 12.0<a>; BurstCurrent = 20.0</a><a> }
                |]
        }
        //...
    |]
```


First I played with loading the catalog from a JSON file - but eventually didn't
see the point in writing all the serialization/deserialization functions.
F# has a very clean data declaration syntax, why not use it?

The downside is that the catalog gets merged into the code - but it sorta
doesn't matter because web browsers will need to download the code + catalog
anyway.


## Application Logic


The application's logic is simple enough to state:

1. 

Users select products for components. Multiple products can be selected
in one component. Whole components can be skipped if the user doesn't
care to choose.
2. 

Designs are produced by finding all the valid combinations of product
selections.
3. 

Stats are generated for each design to help the user choose between them.

From a code stand-point, this boils down to needing to keep a set of
selected products (per category), then writing the design combinator,
then deriving stats.

A sketch of it looks something like:

```csharp
type SelectedProduct =
    {
        ComponentKey : CompKey
        ProductKey : ProductKey
    }

let getDesigns (selProducts : Set) : Design[] = ...
```


This function was not easy to write (80 loc, factored into 8 functions)
and I won't bore you with its
implementation. I will say that it uses F# collections and F# pattern
matching to great effect and I would be hessitant to write that algorithm
in another language. It has to take care of generating combinations of
designs and distributing bundled components - it sounded so easy when
I first started it. :-)

It also calculates stats about the drone using a combination of physics
calculations and data measured from motors.

Unfortunately motor profiles are terribly measured. There are about 4
variables you need to calculate thrust from a motor and online motor
profiles often only provide you with 4 data points. In order to make any
inferences from this terrible data, I had to write fancy math functions
that calculate Jacobians on the fly to do linear extrapolation.
Again, I'm thankful I had F# to
help me through writing that code. Here's a little snippet:

```csharp
let getMaxThrust (v : float) (c : float</a><a>) (d : float) (p : float) (m : MotorModel) : float * float =

    let nearestPoints : MotorModelPoint[] = ...

    let p0 = nearestPoints.[0]

    let diff (fy : MotorModelPoint -> float) (fx : MotorModelPoint -> float) : float = ...

    let dtdv = diff (fun x -> float x.Thrust) (fun x -> float x.Volts)
    let dtdc = diff (fun x -> float x.Thrust) (fun x -> float x.Current_)
    let dtdd = diff (fun x -> float x.Thrust) (fun x -> float x.Diameter)
    let dtdp = diff (fun x -> float x.Thrust) (fun x -> float x.Pitch)

    let t =
        float p0.Thrust
        + dtdv * float (v - p0.Volts)
        + dtdc * float (c - p0.Current_)
        + dtdd * float (d - p0.Diameter)
        + dtdp * float (p - p0.Pitch)
```


Who says you don't get to use calculus in your day to day work?!

So that's about it for application logic. Time for a UI!


## The Reactive UI


The UI is built up using a mix of HTML and custom React classes.
Each React class is backed by an F# View Model object.

The view models are declared as an F# tree rooted at the "app" view model.
This tree gets transformed into the React component tree by the JSX declarations.

Let's look at one of the nodes on that tree. Here is the view model for
the component selectors on the right side of the app:

```csharp
type ComponentView =
    {
        Key : CompKey
        Info : ComponentInfo
        Options : OptionView[]
        Products : ProductView[]
    }
```


This view model is then paired up with a React JSX class (this is JavaScript):

```csharp
var ComponentSelector = React.createClass({
    shouldComponentUpdate: function(nextProps, nextState) {
        return !componentEq (this.props.component) (nextProps.component);
    },
    render: function() {
        var comp = this.props.component;
        var info = comp.Info;
        var products = comp.Products;
        var options = comp.Options;
        return (
            <section><h1>{info.Title}</h1>


            </section>
        );
    }
});
```


That JSX declaration does a lot of things:

1. 

It tests if it even needs to be updated by comparing its old binding
to the new one. Doing these checks drastically improve's React's performance.
In fact, it's the whole reason we're using immutable data structures to begin
with (and, therefore, the whole reason I'm writing this article).
The comparison is done by the `componentEq` global function; more on that later.
2. 

The render function declares the outputted HTML.
3. 

It also continues the mapping process by combining React classes with F#
view models.

It's pretty simple huh? Your UI layer becomes very straight-forward to write.
It's basically all about unpacking variables, choosing some HTML,
and then messing with CSS to get everything to look nice.


## Handling Events


The most important interaction in the app is the user toggling whether a product
is selected. This is handled in the Product React class:

```csharp
var Product = React.createClass({
    handleClick: function(event) {
        setProductSel (this.props.productView.ComponentKey) (this.props.productView.ProductKey) (!this.props.productView.Selected);
    },
    render: function() {
        var pv = this.props.productView;
        var priceEach = pv.PriceEach;
        ...
        return (
            <div>
                <div>
                    <img src="%7Bprod.ImageUrl%7D" alt="image"></div>
                <div>
                    {price}
                    <span>{prod.Name}</span>
                </div>
            </div>
        );
    }
});
```


When a product is clicked, the global function `setProductSel` is called.
Let's take a look at it:

```csharp
let setProductSel ck pk s =
    let k = ck, pk
    if s = TheApp.SelectedProducts.Contains k then ()
    else
        let a = TheApp
        let newApp =
            if s then { a with SelectedProducts = a.SelectedProducts.Add k }
            else { a with SelectedProducts = a.SelectedProducts.Remove k }
        updateAppState newApp
```


where `TheApp` is a global variable of type:

```csharp
type AppState =
    {
        SelectedOptions : Set
        SelectedProducts : Set
    }
```


`setProductSel` is given a component key, a product key, and whether it is
selected. It then recreates the global app state with that new information.
If passes that app state onto the `updateAppState` function:

```csharp
let updateAppState newState =
    TheApp <- newState
    TheAnalysis <- analyze newState
    for l in TheAppListeners do l ()
```


This is, basically, the only mutation in the app. It replaces the global
app state with the new one (I could just as easily have retained it to
create an undo buffer or something.)

It then calculates a new "analysis" which is just the rooted view model tree.

Lastly, it fires off an event to let the UI know that the state has changed.


## The Application Root


I've described how the app runs, but how does it get started?
This is the final bit of glue that merges the React class world with
my F# view model world:

```csharp
var DroneApplication = React.createClass({
    getInitialState: function () {
        var t = this;
        registerAppListener(function () {
            var a = getTheAnalysis();
            window.location.hash = a.LocationHash;
            t.setState ({ app: getTheApp(), analysis: a });
        });
        return { app: getTheApp(), analysis: getTheAnalysis() };
    },
    render: function () {
        var analysis = this.state.analysis;
        var comps = analysis.Components;
        return (
            <div>
                <header><summary>

                    {comps.map(function(c) {
                        return ;
                    })}


        );
    }
});

React.render (
    ,
    document.getElementById('content'));
```


You can see that the first thing the root class does is to register for those
state-updating events. It then returns the global app state (and analysis) as
its own state. When an updated event is fired, it fetches its new state and
invalidates itself.

And that's it! The rest is just writing more view models and more HTML and CSS.


## Generating JavaScript


I have completely ignored the actual process for getting all this code into a
packaged form. I'll try to outline that process now.


### F# Console App


Start by putting all the F# code into an F# console app. This is convenient
because we can "run our app" from the command line to test
our logic or do other wacky things.

It's also necessary to have an *app* and not a library because someone has to
call FunScript to generate JavaScript.

```csharp
[]
let main argv =
    let js = FunScript.Compiler.compileWithoutReturn <@ appMain() @>
    let d = "../Site/build"
    System.IO.File.WriteAllText (System.IO.Path.Combine (d, "client.js"), js)
```


The main entry point for our console app calls the FunScript compiler by
passing it a reference to a function called `appMain`. All code referenced
by `appMain` will end up getting compiled (FunScript has a nice dependency walker).
The console app ends by dumping out the generated JavaScript.

My `appMain` function acts like a standard JavaScript module and exports
a set of functions. Since I'm doing this in the browser, "export" means
that I assign it to the `window` object (it's fine).

```csharp
[]
let external (n : string) (f : 'a) = ()

let appMain () =
    external "registerAppListener" registerAppListener
    external "loadPreviousAppState" loadPreviousAppState
    external "setOptionSel" setOptionSel
    external "setProductSel" setProductSel
    external "getTheApp" (fun () -> TheApp)
    external "getTheAnalysis" (fun () -> TheAnalysis)
    external "analysisEq" (fun (x : AppView) (y : AppView) -> x.Eq y)
    external "componentEq" (fun (x : ComponentView) (y : ComponentView) -> x.Eq y)
    external "designEq" (fun (x : DesignView) (y : DesignView) -> x.Eq y)
```


This `appMain` is perfect because it's easy for me to make F# functions
available to JavaScript and it also satisfies FunScript's dependency checker.
As a final bonus, it's compatible with Google's Closure Compiler.


### It's a Bit... Big


We're doing great, all the F# code has been turned into JavaScript thanks to
the magic of FunScript. But, the code it generates isn't optimal. One might
even say it's unoptimized. It repeats whole expression branches when it doesn't
need to, it loves generating empty expressions, and it does not share generic
implementations.

The 2,500 lines of F# code (1,200 logic + 1,300 catalog) get translated to
**934 KB of JavaScript**... A bit much.

Google to the rescue. Google has a fantastic
[JS compiler called Closure](https://developers.google.com/closure/compiler/?hl=en) that
does all the gross data flow analysis needed to clean out fat code.

I just crank it up to its max settings, pass it the generated code, and out pops
a **168 KB minified file**. Magic.


### My Makefile


Yes, I still use Makefiles. Here's what building the app looks like:

```csharp
OPTIMIZATIONS = ADVANCED_OPTIMIZATIONS

all: public/index.html public/site.js

public/site.js: build/client.js build/components.js Makefile
    java -jar build/compiler.jar --externs react-externs.js --compilation_level $(OPTIMIZATIONS) --js build/client.js --js build/components.js --js_output_file public/site.js

build/components.js: src/components.js
    jsx src build

build/client.js: ../Scraper/Client.fs ../Scraper/Model.fs ../Scraper/Catalog.fs
    xbuild ../DroneBuilder.sln
    mono ../Scraper/bin/Debug/Scraper.exe
```


This file describes the 3 phases of the build:

1. Using mono & FunScript to generate JavaScript
2. Use JSX to generate more JavaScript
3. Using java & the Closure Compiler to smoosh that JavaScript

(The F# conolole app is called "Scraper" - for reasons.)


## Concluding thoughts


And would you believe it, it all works!

I am quite proud of the app. At first it was supposed to be a quick toy
to help me with my hobby, but it quickly became a furtile ground to
try out some new ways to build apps.

I am completely sold on this way to architect apps:

* All data stored in immutable structures
* Mutation localized to very specific points
* Generating light-weight view hierarchies from that data

While I sometimes miss VB and mutating all the things - I don't miss the
bugs.

Apps written in the functional style are easier to write, easier to
understand later, and easier to extend to new scenarios.

There are tradeoffs of course. Functional languages and libraries are great
at handling trees but they suck at graphs - and I find most apps to be graphs.


### And then there's FunScript


I love FunScript, it's one of the best transpilers I've ever used, but I
don't think I'll ever use it again.

The problem is that it just doesn't do any optimization and ends up generating
code that JavaScript just can't handle. For instance, Drone Builder is very slow
when you first start clicking around using an iPhone (it's fine on Desktop
browsers). It takes a long time for
the browser to JIT all the methods it needs to to make the app run fast.

On top of that, the error reporting in FunScript is horrendous. I love getting
errors like "never" and "interface not found" with absolutely no indication
which line of code triggered this bug.

I gave up on this project once because I couldn't understand one of these errors
and didn't know what to change. (Finally I got lucky and changed the right thing.)
Then it happened again towards the end of the project when it refused to compile
equality comparisons.

Now equality comparisons are one of the main reasons I'm using functional data
types. It was a real blow but I pushed on and wrote my own equality comparisons
(had gone too far to give up).

These types of problems are to be expected with a project like FunScript - bugs
happen. The real crux thogh is that the maintainer of the project hasn't worked
on it in a while and is not interested in continuing work. SO I'm not seeing
a bright future of these bugs getting fixed.

The good news though is that this app has validated this style of programming.
I just have to work on what tools I use to achieve it.

