---
layout: post
title:  "Automated UI Testing with Cucumber and Calabash"
date:   2014-01-17 23:02:34 GMT
redirect_from:
  - /post/73655359097
  - /post/73655359097/automated-ui-testing-with-cucumber-and-calabash
tags: article
---



There comes a day in every developer’s life when they have to admit that they just aren’t good testers. We don’t think to hit buttons in strange combinations, we test features in isolation, we don’t re-test for regressions, and we simply don’t do it often enough.

Thankfully most of us realize this early in our careers. We adopt unit tests to make sure our functions work (at least in isolation). We worship continuous integration servers. We annoy friends to test our apps and watch in horror as they hit submit buttons ten times in rapid succession. (A week later, you ask them to try again. “Ha! I won this round!” you gloat.)

If you’re lucky enough to work at a company with dedicated testers, you even get to substitute “colleague” for “friend”.

But what if your friends just aren’t cutting it? Or what if we cared about them enough to realize that clicking every button in the app on every release is just a new form of torture so far unidentified by the US government?

This article will introduce you to [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin) a language for specifying tests, [Cucumber](https://github.com/cucumber/cucumber) the tool that runs those tests, and [Calabash](http://calaba.sh) the library that interfaces Cucumber with the platform you’re testing. This article will use an iOS app ([Calca](http://calca.io)) written with Xamarin.iOS as an example, but it is mostly platform agnostic.

It is my goal to show you that this environment is easy to setup, that it’s “not-so-hard” to script UIs, and that using this setup will make your apps more reliable. You will certainly relieve some of the burden placed on those you’ve roped into testing.


## Automating the UI


We wish to automate testing the user interface of our applications. This will, inevitably, boil down to tapping on the screen, waiting for the app to respond, then validating that response.

We specify these steps using a language called [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin) that codifies these phases of testing using natural language.

To demonstrate this natural language, here is a series of steps I use to automate document renaming in [Calca](http://calca.io) using Gherkin:

```csharp
Scenario: Renaming a document
  Given a new document
  When I touch the title
  Then I should see "Rename"

  When I enter "A New Name" into the "Rename" input field
    And I touch the "Done" button
  Then I should see "A New Name"
    And I delete the open document
```


Yes, this is a scripting language, and a pretty clever one too.

Steps are grouped into **Scenarios** which are like scripts but don’t have variables or branching. We can see that my scenario is called *Renaming a document*.

Each step is specified as a **Given**, a **When**, or a **Then**:

* **Given** sets up the preconditions of the test
* **When** specifies an action the user takes or acknowledges an event
* **Then** validates the state of the app

**And**, **Or**, and **But** can also be used to make the steps more readable. (The truth is that all these labels are ignored by the test runners and are there simply to make your tests more readable. Use them.)

You will notice that my script is pretty self explanatory because it is written in a formalized English. I’m sure I could hand it to any non-programmer and they could comprehend it and even validate its correctness.

Imagine an entire app’s behavior described in the language of scenarios and steps. It would be a lot like a functional spec for the app. Perhaps we don’t want fall down the slipperly slope of overspecification, but there are certainly benefits to a written and always up to date description of an app’s behavior. For example, these steps are very close to being user documentation - perhaps we can have verifiable documentation too.

What if you wrote these scenarios before you wrote the app’s code? This is like Test Driven Development (TDD) where you write functional tests before writing the code that makes it pass. Since we are at a higher level of usage, this style is called **Behavior Driven Development** (BDD).

The BDD process goes something like:

1. Imagine a new feature
2. Document the feature as a set of user scenarios needed to exercise that feature
3. Code and code until all the steps of #2 pass

Cucumber, the tool that will execute these scenarios, is designed to handle this workflow and is a pleasure to use. It even handles executing undefined steps gracefully going so far as to stub out the code needed to define them.


### Predefined Steps


Calabash comes with a variety of [predefined steps](https://github.com/calabash/calabash-ios/wiki/02-Predefined-steps). These include:

```csharp
When I touch "accLabel"
When I scroll down
Then I wait to see "text or label"
Then take picture
```


among many others.


### Defining Steps


You will, eventually, want to write your own steps. You’ll do this for a variety of reasons:

* A part of your UI is hard to distinguish with the predefined steps
* You want to abstract out elements or interactions with the UI (for platform differences)
* You want to reuse code between scenarios
* You want to change the step dialect

Steps are defined in a Ruby DSL (yes, that [Ruby](https://www.ruby-lang.org/)) with a template sentance regular expression and a script body. The script body will be executed when the step is executed.

For example, the step `And I touch the "Done" button` is implemented with this Ruby DSL:

```csharp
Then /^I (?:press|touch) the "([^\"]*)" button$/ do |name|
  touch("button marked:'#{name}'")
  sleep(STEP_PAUSE)
end
```


Notice the use of the regular expression that matches the step’s natural language. Also note that the regex has a grouping that is mapped to the `name` parameter of the body of the script. This makes writing parameterized steps easy.

This script itself makes use of the Calabash library to interface with iOS. The `touch` function queries the view hierarchy for a `UIButton` with an `accessibilityLabel` (marked) with the `name` specified in the step. In our case, it will look for a button with the label “Done”. The sleep function is run to throttle Cucumber - we have to give those animations time to finish after all.

To get a feel for step definitions, I recommend reading the reference to the [standard steps provided by Calabash for iOS]() while simultaneously reading [the source code to those steps](https://github.com/calabash/calabash-ios/blob/master/calabash-cucumber/features/step_definitions/calabash_steps.rb).

You will find out that you need to learn two libraries in order to write step definitions:

* [Calabash for iOS Ruby API](https://github.com/calabash/calabash-ios/wiki/03.5-Calabash-iOS-Ruby-API)
* [Calabash View Query Syntax](https://github.com/calabash/calabash-ios/wiki/05-Query-syntax)

With this knowledge you can leverage Calabash’s built in steps and your own custom steps to write descriptive and useful scenarios.


### A Balancing Act


Let us look at the rename scenario and note where each step comes from:

* Given a new document - **App-specific**
* When I touch the title - **App-specific**
* Then I should see “Rename” - **Calabash**
* When I enter “A New Name” into the “Rename” input field - **Calabash**
* And I touch the “Done” button - **Calabash**
* Then I should see “A New Name” - **Calabash**
* And I delete the open document - **App-specific**

Three of the steps were written by me, while the other 4 are predefined Calabash steps.

I have found there to be a balancing act between writing your own steps and using the predefined ones.

On one hand, the [predefined steps](https://github.com/calabash/calabash-ios/wiki/02-Predefined-steps) are easy to write and are general enough to be useful.

Unfortunately, your scenarios can end up with a lot of duplicated code as features tend to get buried a few clicks away and usually involve some cleanup.

I defined the `Given a new document` step because I found myself repeatedly writing these Calabash steps to create a new document:

```csharp
Given I touch "Add"
  And I touch the "New Calculation" button
  And I touch "Done"
```


These three lines of code got repeated in a lot of scenarios because I liked to test when there was a new document available.

Eventually, I wisened up. Instead of repeating these lines, I wrote a new app-specific step in Ruby:

```csharp
Given /^a new document$/ do
  macro "I touch \"Add\""
  macro "I touch the \"New Calculation\" button"
  macro "I touch \"Done\""
end
```


I was able to take advantage of the `macro` function to write this custom step in terms of other steps.

Now I am able to write `Given a new document` in my scenarios instead of those three Calabash steps. It’s a win.

Custom defined steps can be useful to abstract parts of UI in order to smoorthly adapt as the application’s UI changes. That is, they can be used to decouple the test from the UI. However we must also be weary of our tests getting too abstracted from the UI. We might end up testing our own tests then.


## Exploring Your App


Calabash relies on accessibility labels and classes to identify views in your application (see the [query syntax](https://github.com/calabash/calabash-ios/wiki/05-Query-syntax) reference). This is far more reliable than identifying views by location. It also means you have to take a bit of care when writing your apps to make sure all interesting views have good accessibility labels.

If your app is all buttons and text fields, everything will just work as you expect.

But chances are, your app is more complex than that. Fortunately Calabash comes with an excellent tool for exploring your application’s views and verifying that you’re setting accessibilty labels correctly.

Once you have installed everything, you will have a tool called `calabash-ios console`. This tool gives you an interactive Ruby session with the [Calabash API](https://github.com/calabash/calabash-ios/wiki/03.5-Calabash-iOS-Ruby-API) ready to be accessed.

All you have to do is run your app (that is configured to talk to Calabash) in the iOS simulator, run this console, and then start typing queries.

For example, you might want to see all visible views:

```csharp
query("view")
```


This will give a list of views, their classes, the screen positions, and the accessibility labels.

From here it’s just a matter of playing with your UI and discovering which queries identify the views you intend.

For example, I might look for a button with the label “Done”:

```csharp
query("button marked:'Done'")
```


If, however, I didn’t use a UIButton, but instead implemented my own class, I might try a different query:

```csharp
query("view:'MyApp.CoolButton' marked:'Done'")
```


This would find all the CoolButtons visible in your app (in the namespace MyApp) with the label “Done”.

To speed up exploration, there is a short-cut query called `labels` that just returns the views’ labels. For example, you can type:

labels(“button”)

to get a list of all the labels of all the UIButtons visible on the screen.


### Fixing Labels


You will find that it is hard to find a query that uniquely identifies a view if it does not have a good label.

If your app uses custom UI elements or other fancy trickeries, chances are your accessibility labels are not nicely set.

As an additional incentive to having good labels, iOS makes use of these same labels to provide VoiceOver for guided screen access. Your app will be easier to use if you get all this right. For iOS, read the [Crafting Useful Labels and Hints](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/iPhoneAccessibility/Making_Application_Accessible/Making_Application_Accessible.html#//apple_ref/doc/uid/TP40008785-CH102-SW6) section of the Accessibility Programming Guide.

All views that implement the [UIAccessibility Protocol](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIAccessibility_Protocol/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008786) have an `accessibilityLabel` property that can be easily set.

To set the label in Xamarin.iOS, you simply write:

```csharp
playButton.AccessibilityLabel = "Play";
```


If you inherit from a class that does not already implement UIAccessibility, then you will need to export this property:

```csharp
[Export("accessibilityLabel")]
public string AccessibilityLabel { get { return "Play"; } }
```



## Getting Started


The remainder of this article gives an overview of what you need to start writing UI automation tests for your own app.


### Install Calabash and Cucumber


Calabash documents the [installation procedure](https://github.com/calabash/calabash-ios) in their README.

Roughly, you will use Ruby’s `gem` package manager to install Calabash and Cucumber on your system.

```csharp
$ sudo gem install calabash-cucumber
```



### Instrument Your Application


To interact with your app, Calabash uses an internal web server that your app must launch.

For Xamarin.iOS, this is rather easy to setup. Just reference the Calabash component from the [Xamarin Component Store](http://components.xamarin.com) and add this line to your AppDelegate’s `FinishedLaunching` method:

```csharp
#if DEBUG
Xamarin.Calabash.Start();
#endif
```


Calling the `Start` function is all you need to do.

You want to conditionally compile that call because Calabash uses evil tricks to get its job done. Apple will not let you release an app with that kind of trickery so you want to control its existence.

Personally, I recommend using another definition such as `CALABASH_INSTRUMENTATION` because you might actually want to test against release binaries. But `DEBUG` is what appears in all the docs.


### Create Your Cucumber Features


The next step is to start writing scenarios. We begin by using a tool to scaffold all the files needed to run the scenarios. Invoke this command in your iOS project directory:

```csharp
$ calabash-ios gen
```


This will create a subdirectory `features` with this hierarchy:

```csharp
features/
  step_definitions/
    * Ruby step definitions (*.rb)
  support/
    * Don't touch anything in here
  * Gherkin feature definitions (*.feature)
```


What is a feature? It’s just a group of scenarios. For example, I might have this feature:

```csharp
Feature: Rename
  ...
  Scenario: Renaming to a new name
    ...
  Scenario: Accidentally trying to rename over another doc
    ...
  Scenario: Renaming to the same name
    ...
```


These three scenarios are written in a single file called `rename.feature` that is stored in the `features` directory. All scenarios must be specified in a feature file.

Custom defined steps get placed in Ruby files in the `step_definitions` directory. I like to have 2 files there, one that is shared between all my apps, and one that contains steps specific to the app being tested.

For Calca, I have:

```csharp
shared_steps.rb
calca_steps.rb
```


in `features/step_definitions/`.


### Running Cucumber


Cucumber will run all the features and all their scenarios when executed.

```csharp
$ cucumber
```


It detects which app to execute based on the current directory (which should be your project directory, not the features directory) and runs the version already installed on the Simulator.

It will execute each scenario as a separate invocation of your app on the simulator. That is, your app will be closed between each scenario and brought back up.

Cucumber has a nice display (for a console app) so make sure your terminal has ANSI colors enabled and is nice and wide.

You can control the simulator parameters by setting environment variables before executing cucumber.

```csharp
$ SDK_VERSION=6.0 DEVICE=ipad cucumber
```


You can also run specific features by passing them as arguments to Cucumber.

```csharp
$ cucumber features/rename.feature
```


[Running on devices](https://github.com/calabash/calabash-ios/wiki/07-Testing-on-physical-iDevices) is also supported.


## Conclusion


I worry a lot when I release an app. Did I test it thoroughly, did I hit every button like a monkey? I will freeze code and just test for days leading up to a release. It’s a very stressful time; every little bug fix can accidentally break something else.

And so, every small fix leads to a full end-to-end testing of the app. But the truth is, that’s impossible to achieve. Inevitably a mistake gets through, a regression introduced at “some point”.

Calabash, Cucumber, and Gherkin relieve the stress of regressions. If I simply take the time to write a feature that excercises new code through a few scenarios, then I can worry a lot less about introducing regressions.

I can run the full test suite any time. I can run it on a variety of devices. I can compare different versions of the app running on different operating systems. I can wire it into my continuous integration server and always know the state of my apps even as platforms evolve. Bye bye bitrot.

Finally, I can stop bothering my friends. I can free them from the mundane testing to do what they do best - breaking my apps in new and inspiring ways (and I can code those into scenarios later).
