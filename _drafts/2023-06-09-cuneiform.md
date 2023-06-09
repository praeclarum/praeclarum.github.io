---
layout: post
title:  "I Built the World's Largest Translated Cuneiform Corpus"
thumbnail: "/images/2023/webgpu-torch_thumb.png"
tags: article
---

**TL;DR** I used a custom-trained Large Language Model (T5) to
create the world's largest online corpus of translated cuneiform
texts. It's [available now](https://cuneiform.pages.dev) and contains
over 30,000 translated texts from the [CDLI](https://cdli.ucla.edu/)
and [ORACC](http://oracc.org/) projects.

## Cuneiform

Cuneiform is the oldest known writing system. It was used in
Mesopotamia (modern day Iraq) for over 3,000 years. It was used to
write Sumerian, Akkadian, and other languages. Written on clay,
it has survived the millennia and is now being translated by
scholars around the world.

Sadly, we have more clay tablets than scholars. Fortunately,
we have computers.

## A Need for Machine Translations

While I am not a cuneiform expert, I am an expert at neural networks
and have a deep passion for languages and writing systems.

Nine months ago I decided that I was in a unique position to make a large
contribution to the world of cuneiform translations. While there are
existing online repositories that contain many transliterations
(a transliteration is simply a rewriting of a text from one writing system
to another without changing the language), they are very lacking
in the translation department.

For example, consider Sumerian (spoken by the builders of the zigurauts).
There are currently 116,060 texts published. 104,151 of these have
been transliterated from cuneiform symbols to a jumble of latin letters.
But only 6,636 of these texts have publicly available translations online.
That is a mere 6% of texts available to a lay person such as myself. 

#### Sumerian

|Publications|Count|
|---|---|
|   Transliterated| 103,075|
|       Translated| 4,583|
|Need Translations| **98,492**|

Given the existing transliterations, there are 98,492 works
that can be translated but have not yet been.

(There are more translations than these, but the others are not freely
available and are held under copyright. In other words, you need to go 
by a book to read them.)

Things aren't much better for Akkadian (the language spoken by the famous Sargon
and Ashurbanipal).

#### Akkadian

|Publications|Count|
|---|---|
|   Transliterated| 31,747|
|       Translated| 10,069|
|Need Translations| **21,678** |


We can see that 21,678 works are all set to be translated but have
not been. 

I decided that it was time to unleash
the power of the machines and make my own contribution to Assyriology. My goal was to publish the world's largest corpus of
translated cuneiform texts. I want any person to have access to the archives of
the ancients. A grandiose goal for sure, but also a very achievable one thanks to
modern engineering advancements.

## Large Language Models

The modern advancement of large language models (LLMs) has affected and
will continue to affect nearly every human endeavor.

The current architecture that is heralding this new age of knowledge is the humble Transformer architecture. It was designed specifically to be very good at translating text from one language to another using the innovative "attention mechanism". It's a little funny that this network designed for translation is now broaching the realm of artificial general intelligence (AGI), but I digress.

Ignoring the absurdly large LLMs that are dominating the field now (GPT-4 and friends), the humble smaller transformers are still quite powerful and have made the problem of translation a somewhat trivial.

My favorite one of these is the T5 network from Google. While large itself, it is capable of being trained using off-the-shelf (though expensive) GPUs. If you can build a large training set, you can 
train this network at home to accomplish wonders.

Knowing this I set about building a training
set that the network could use to learn 
these ancient languages.

## Building the Dataset

Thankfully there has been a push to digitize acquired artifacts and to publish their cuneiform on the web.

The two great projects are the CDLI (Cuneiform Digital Library Initiative) and ORACC (Oriental XXX). I owe a large debt
to these projects.

As any machine learning expert will tell you, 90% of the problem is collecting a good training dataset (the other 10% is justifying the compute bill). Building the cunefirom datset presented its own uniqe set of challenges.

### Inconsistent Transliterations

Sadly, Assyriologists took some time to settle on a consistent transliteration system. When works were first transliterated to a digital form, only ASCII characters were available and the researchers made due using funny characters like # to denote demonstratives, numbers to disambiguate symbols, and ALL CAPS whenever they were in the mood (just kidding, but the use is so random it might as well be).

When other character encodings became available, researchers adapted. They started to use diacritic marks to disambiguate symbols (loosely based on guessed sounds). And then HTML was invented and they went wild with special marks attempting to better capture the original writing.

While neural networks are powerful and can certainly handle these inconsistencies, it's not ideal. If you want the network to properly learn the language it's best not to distract it with also learning the histrionics of human computer interface systems.

### Languages Change

Languages change over time. The language I speak compared to my parents is already different and that is a mere 30 difference in ages. Sumerian was used for millennia. It changed. Even after being supplanted by Akkadian, it continued to change as scribes from Akkad would keep Sumerian words alive by incorporating them into Akkadian texts.

### Learning Sumerian and Akkadian Simultaneously

### Long and Short Translations

