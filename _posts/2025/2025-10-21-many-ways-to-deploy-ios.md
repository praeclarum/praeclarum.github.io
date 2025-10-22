---
layout: post
title:  "The Many Ways to Deploy iOS Apps in 2025"
thumbnail: "/images/2025/deploy_ios_thumb.jpg"
tags: article
---

**TL;DR** There are no less than 4 different ways you can deploy your iOS app to a physical device for testing. I enumerate all of them below, along with their pros and cons. In the end, I recommend using `dotnet build -t:Run` if you can, as it is the simplest and most reliable method. But knowing the alternatives can be useful in certain situations.

## From None to Many

Just a couple years ago, I didn't know *any* of the ways to deploy my .NET/MAUI iOS apps to a physical device. I only knew how to use Visual Studio for Mac's built-in run button, which worked fine for me. Until they killed it.

At present, there are only two IDEs for macOS that support .NET MAUI iOS development: JetBrains Rider and Visual Studio Code. Rider *should* be nice, but has terrible bugs where it will sometimes try to deploy `iossimulator` builds to the device, or that persistence bug where it fails to rebuild apps when dependencies change, or it just doesn't detect devices at all. It's not at all reliable. Visual Studio Code *should* be nice too, but it basically requires that you write MAUI apps for anything to work. If you're like me and prefer to build native UIs, then you're out of luck.

But there is good news, and I am here to deliver it. There's not one but *four* different ways you can deploy your iOS app to a physical device for testing, none of which require an IDE. Here they are, in order of my preference with my own heart-felt pros and cons.

## 1. `dotnet build -t:Run`

It turns out, good old MSBuild has a built-in task to do exactly what we want. You can use the `dotnet build` command with the `-t:Run` target to build and deploy your app to a connected device in one step. How is this different from `dotnet run`? I don't know. Don't ask me, ask Microsoft.

You can read all about `dotnet build -t:Run` in the official ["Launch the app on a device" documentation](https://learn.microsoft.com/en-us/dotnet/maui/ios/cli?view=net-maui-9.0#launch-the-app-on-a-device).

Just run:

```bash
dotnet build -t:Run -f net9.0-ios -p:RuntimeIdentifier=ios-arm64 -p:_DeviceName=MY_SPECIFIC_UDID MyApp.csproj
```

and replace `MY_SPECIFIC_UDID` with your device's UDID and `MyApp.csproj` with the name of your project file (or elide it if you're in the project's directory). Adjust the `-f` target framework to match your project (for example, `net8.0-ios`).

A UDID is a Unique Device Identifier, a unique string that identifies your iOS device. It's like a UUID, but there's a D instead of a U.

### How to Find Your Device's UDID

Don't you worry, here are 4 easy ways to find your device's UDID:

1. **Xcode**: Open Xcode, go to "Window" > "Devices and Simulators", and select your device. The UDID will be listed there.
2. **xcrun**: Buried in every Xcode installation is the `xcrun` command line tool. You can use it to list connected devices and their UDIDs with this command:
    ```bash
    xcrun devicectl list devices --hide-default-columns --columns Name --columns UDID
    ```
    Output will look something like this:
    ```
    Name                    UDID                     
    ---------------------   -------------------------
    Precious XXIII          00008030-000E409C0E10802E
    Precious XXIV           00008103-001829AE14BB001E
    Precious XXVII          00008140-000A0DC83013C01C
    Precious XXVIII         00008150-001C2C3E0EFB801C
    ```
    The only problem is that it will only report modern devices with their modern UDIDs. If you have devices running iOS 15 or earlier, Apple doesn't support them with `devicectl`, so you'll have to use one of the other methods.
3. **mlaunch**: Buried so deeply in the .NET iOS workloads that you'll need an oxygen mask to find it, is another wonderful tool called `mlaunch`. You can use it to list connected devices and their UDIDs with this command:
    ```bash
    mlaunch --listdev
    ```
    Output will look something like this:
    ```
    Precious XXIII: 00008030-000E409C0E10802E
    Precious XXIV: 00008103-001829AE14BB001E
    Precious XXVII: 00008140-000A0DC83013C01C
    Precious XXVIII: 00008150-001C2C3E0EFB801C
    Precious XVIV: c52a6fd19cc179aad6696abe67cce53705bf22d0
    ```
    (along with a bunch of errors). This works for all devices, old and new. Don't make fun of the XVIV, Roman numerals are hard.
4. **ios-deploy**: Thanks to a few saints masquerading as software developers, there is the `ios-deploy` tool available one agonizing `brew install ios-deploy` away. You can use it to list connected devices and their UDIDs with this command:
    ```bash
    ios-deploy --detect
    ```
    Output will look something like this:
    ```
    [....] Found 00008103-001829AE14BB001E (J517AP, iPad Pro 3G (11"), iphoneos, arm64e, 26.0, 23A340) a.k.a. 'Precious XXIV'
    [....] Found c52a6fd19cc179aad6696abe67cce53705bf22d0 (D11AP, iPhone 7 Plus, iphoneos, arm64, 15.7, 19H12) a.k.a. 'Precious XVIV'
    ```
    This works for all devices, old and new *so long as they are available* for deployment. You see, the other tools report devices that are paired to the host machine, even if they are not available for deployment (e.g. locked, no trust relationship, etc.) but `ios-deploy` only reports devices that are actually available for deployment. This can be a blessing or a curse depending on your mood.

## 2. `mlaunch`

Buried so deeply in the .NET iOS workloads that you'll need a spatula and a crowbar to find it, is the wonder aptly named `mlaunch`. You can use it to deploy your app to a connected device in two steps: first install the app, then launch it.

There is no documentation for `mlaunch`, but if you run it with the `--help` flag, you can see the myriad of options available to you. We shall discuss the most mundane ones here.

To install your app, you're first going to need to build it. You can do this with the following command:

```bash
dotnet build -f net9.0-ios -r ios-arm64 MyApp.csproj
```

Now you can install the app with this command:

```bash
mlaunch --installdev=bin/Debug/net9.0-ios/ios-arm64/MyApp.app --devname=MY_SPECIFIC_UDID
```

The location of mlaunch is curious, spurious, and mysterious. You can find it where all the great workloads are hidden, I mean stored. Somewhere beneath `/usr/local/share/dotnet/packs/Microsoft.iOS.Sdk.*`

For example, on my machine it is located at: `/usr/local/share/dotnet/packs/Microsoft.iOS.Sdk.net10.0_18.5/18.5.10727-net10-rc.1/tools/lib/mlaunch/mlaunch.app/Contents/MacOS/mlaunch`

Now that your app is installed, you can launch it with this command:

```bash
mlaunch --launchdevbundleid=MY_BUNDLE_ID --devname=MY_SPECIFIC_UDID
```

Replace `MY_BUNDLE_ID` with your app's bundle identifier (e.g. `com.mycompany.myapp`) and `MY_SPECIFIC_UDID` with your device's UDID.

That's it, you're now a pro `mlaunch` user!

## 3. `xcrun`

If you are not in the mood to dig through the .NET SDK installation to find `mlaunch`, you can use the `xcrun` command-line tool that comes with Xcode. It has a subcommand called `devicectl` that you can use to install and launch your app on a connected device.

Build your app, and then run this command to install it:

```bash
xcrun devicectl device install app --device MY_SPECIFIC_UDID bin/Debug/net9.0-ios/ios-arm64/MyApp.app
```

Then run this command to launch it:

```bash
xcrun devicectl device process launch --terminate-existing --console --device MY_SPECIFIC_UDID MY_BUNDLE_ID
```

The `--terminate-existing` flag will kill any existing instance of your app before launching it, and the `--console` flag will stream the app's console output to your terminal. Hot.

Sadly, `xcrun devicectl` does not support older devices with long-in-the-tooth iOS versions, so if you have an iOS 15 or older device, you should go spelunking for `mlaunch` instead.

You've made it this far, you are truly a command line iOS hacker dev. But there's one more way to deploy your app, and it's the most hacky way of them all.

## 4. `ios-deploy`

If you aren't satisfied with first- or third‑party tools, and want to dabble with “fourth‑party” greatness, then you can use the `ios-deploy` command-line tool. This tool is not officially supported by Apple or Microsoft, but it is widely used by the iOS development community.

First, make sure you have `ios-deploy` installed. If you haven't done this yet, you can install it using Homebrew:

```bash
brew install ios-deploy
```

Make sure you have a full battery, a gigabit internet connection, and a giant cup of coffee, because this may take a while. `ios-deploy` is small and quick to install, but Homebrew has to download 1/2 of the internet and at least 3 different versions of Python before it will let you have your machine back.

Once you have `ios-deploy` installed, you can use it to deploy and run your app. Build your app, then run:

```bash
ios-deploy --justlaunch --debug -i MY_SPECIFIC_UDID -b bin/Debug/net9.0-ios/ios-arm64/MyApp.app
```

And that's it! You've now deployed your app using `ios-deploy`. Check out the [ios-deploy README on GitHub](https://github.com/ios-control/ios-deploy) for more information and advanced usage.

## Conclusion

There you have it, four different ways to deploy your iOS app to a physical device for testing. No IDE needed. My personal favorite is `dotnet build -t:Run` because it's the simplest and most reliable method. But knowing the alternatives can be useful in certain situations, when showing off to your mom, looking cool at parties, or when you just want to feel like a true command‑line iOS hacker dev.
