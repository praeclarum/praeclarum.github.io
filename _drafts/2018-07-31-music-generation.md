---
layout: presentation
title:  "Music Generation on iOS"
thumbnail: "https://img.youtube.com/vi/KCoEqOG1OdU/0.jpg"
tags: article video announcement
code: "https://github.com/praeclarum/NMusic"
youtube: KCoEqOG1OdU
---

Four years ago I was struggling with insomnia and wrote an app to generate lullabies.
I never did get to sleep, but I did end up writing a fun music generation
library in C#.

This post will introduce you to the library so you can add a 
bit of music to your own apps.


## Playing Instruments

The library contains a functional `Instrument` object for controlling (playing notes on) a MIDI instrument. You can create these and play notes using async methods (that return when the note completes).


## Music Theory

You can control instruments yourself, or you can create a `Song` object that will generate a random fully instrumented song.

It does this by defining basic musical constructs such as `Scales`, `Keys`, `Chords`, and `Progressions`.

For instance, here are some basic chords:

```csharp
public static readonly Chord Major = new Chord ("maj", 0, 4, 7);
public static readonly Chord Minor = new Chord ("min", 0, 3, 7);
public static readonly Chord Augmented = new Chord ("aug", 0, 4, 8);
public static readonly Chord Diminished = new Chord ("dim", 0, 3, 6);
```

where each chord has a name and series of subintervals - these are the notes to play on a scale once a tonic (root tone, first note of the chord) has been chosen.

There is a full list of scales though you probably won't use most of them:

```csharp
public enum Scale
{
    Major = Ionian,
    Minor = Aeolian,
    Ionian,
    Dorian,
    Phrygian,
    Lydian,
    Mixoldian,
    Aeolian,
    Locrian,
}
```

Given a `Chord`, a `Scale`, and a tonic, the notes of the chord can be calculated. These notes can then be played on an `Instrument`
to produce nice sounds.

But things only get interesting when we start playing different chords at different times. Chords are composed together into `Progressions` that last for a certain number of beats. A small database of common progressions is provided with entries such as:

```csharp
public static Progression Major1 = new Progression {
    { 1, Chord.Major, 2 },
    { 3, Chord.Minor, 2 },
    { 6, Chord.Minor, 2 },
    { 2, Chord.Minor, 2 },
    { 5, Chord.Major, 2 },
};
public static Progression PopPunk = new Progression {
    { 1, Chord.Major, 2 },
    { 5, Chord.Major, 2 },
    { 6, Chord.Minor, 2 },
    { 4, Chord.Major, 2 },
};
public static Progression Turnaround = new Progression {
    { 1, Chord.Major, 4 },
    { 2, Chord.Minor, 2 },
    { 5, Chord.Major, 2 },
    { 1, Chord.Major, 2 },
};
```

This gives a bit of structure to compositions but I think still gives a lot of freedom to play around in.


## Composing Songs

Finally, the library defines the root `Song` class that is made up `Sections` (like verse, chorus, bridge). Each `Section` contains a `ChordProgression` and a tempo (BPM). `Melodies`, `Baselines`, etc. are generated for each section.

