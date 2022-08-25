---
layout: post
title:  "Transformers-js - Neural Networks in the Browser"
thumbnail: "/images/2022/hard_problems_thumb.jpg"
tags: announcement
---

**TL;DR** I wrote a javascript library that lets you run modern transformer neural networks from Hugging Face 🤗 in the browser. It works on mobile browsers, desktop browsers, pretty much everywhere. Check out [transformers-js on GitHub](https://github.com/praeclarum/transformers-js) to see how it works and checkout out the live translation demo running on my static website: [https://transformers-js.praeclarum.org](https://transformers-js.praeclarum.org).


## Introduction

Currently, the best way to deploy neural networks is to pay a cloud provider to host it and pay them to run inference. The more customers you have, the more you pay. It’s an old-fashioned big-iron middle-man’s utopia.

I’m a big fan of running neural nets on everyday hardware. It makes sense to let customers, who already invested a lot of money and carbon, use their own hardware. It’s also a huge privacy win: attackers can’t steal your information if it’s never on the network (insert Intel joke here). It’s good economically, environmentally, and it’s good for security. Sign me up.

Let’s fight the big-iron trend. Let’s run neural networks in the browser!

Announcing [transformers-js](https://github.com/praeclarum/transformers-js): a library to make running translation and other language neural nets in the browser simple.


## Hugging Face 🤗 Transformers with transformers-js

Transformers are neural networks that are good at manipulating serialized symbols. Ahem, sorry. By “serialized symbols” I mean language. They do language things: Sentiment analysis, summarization, translation, transmogrification. Basically, any -ation you can think of that works with a discrete set of symbols laid out one after the other.

And you know these networks from their friendly household names: GPT-3, Copilot, DALL-E, Stable Diffusion. There seems no end to what they can do (see also the CNN revolution of 2014).

[Hugging Face 🤗](https://huggingface.co) has established itself as the “GitHub of Transformers”. They have an excellent unifying framework, great documentation, and good-ish hosting. I only say good-ish hosting here because I had a demo fail because their servers were down. Clouds…

In fact, it was that demo fail that got me to thinking, “why can’t I just run this thing in the browser?” That thought led me to 3 days of programming. Those 3 days produced a javascript library. And that javascript library produces some kick-ass neural translations.

I wrote transformers-js to make running transformers from Hugging Face 🤗 in the browser just as easy as running them in Python land. To do this, I leverage the amazing ONNX runtime in order to run the network. ONNX offers a browser-compatible runtime using WASM compiled from the complete ONNX opset code. That’s very powerful because it means that, if you can get your net running in ONNX, you can get it running in the browser. (ONNX also offers a webgl backend that is much faster than their WASM backend. But you lose so much precision in webgl that I have yet to see a network work correctly using that engine.)

But running the neural network is only half the battle. Running transformers requires more software than just the neural net. You also need text tokenization software to convert your text to tokens (symbols) and you need sampling software to convert the neural net’s output probabilities back to symbols. Transformers-js takes care of all that for you. 


### Tokenization

Step 1 in running a transformer is getting a working tokenizer. Each neural net is optimized to solve a problem and that means each net uses a slightly different tokenizer from each other.
I thought writing the tokenizers would be a piece of cake. I’ve written hundreds of tokenizers in my career in my pursuit of programming language nirvana, but I have never run into the kind of tokenizers that data scientists have come up with.

**Side tangent:** did you know that modern tokenizers use classical AI approaches? Neither did I! For example, the T5 symbol list is redundant; you can encode the same sentence many many different ways. In order to correctly tokenize the sentence for input to T5, you have to find the optimal path through the redundant symbol list based on the a-priori probabilities of the symbols. It’s a graph problem, and those are hard. Fortunately, classic AI people loved graph problems and found solutions. Two AI winters ago, people thought graphing techniques would be the foundation of all future AIs. They were wrong, but it’s nice to see these old powerful algorithms live on.

Back to tokenizers. I learned all that graph theory so you don’t have to! I encoded that knowledge into code that a computer can decode to make the magic happen. Behold:

```js
// Load the tokenizer
const tokenizer = await AutoTokenizer.fromPretrained("t5-small", "/models");
```

That loads a tokenizer. Currently, I only support Sentence Piece Unigram models (good enough for most nets). I hope to support Byte Pair Encoding in the future (GPT’s preferred tokenization).

With that tokenizer, you can convert strings into symbol lists:

```js
// Tokenize "Hello, world!"
const english = "Hello, world!";
const inputTokenIds = tokenizer.encode("translate English to French: " + english);
```

`inputTokenIds` is a list of integers that represent the symbols in the sentence. Some words are just one symbol. While other, less common or longer words, can be more than one symbol.

I added a little prefix to the string (“translate English to French:”) because I’m building up to a translation demo here and the T5 network, with all its advanced capabilities, needs to be told what to do.


### Generation

Now that we have tokens, we can hand them off to the neural network to be run:

```js
// Translate
const outputTokenIds = await model.generate(inputTokenIds, {maxLength:50,topK:10});
```

That's it! The code takes the input tokens, runs them through the network, and returns a new list of output tokens.

That little `generate` function is hiding a lot of work. Most networks generate one token at a time. That means you have to
run them over and over until you get the whole sentence. This can be terribly inefficient if you run the *entire* network
over and over. Instead, you split it into pieces and run each piece only as it is needed.

The `generate` method also has to sample from the neural network’s output probabilities. Networks are not into commitment, and will always output a variety of options. A sampling technique is needed to pick the right one.

*Greedy* sampling is when you just pick the highest probability option. *Top-k* sampling is when you randomly pick from the top `k` probable options. Greedy is good for when you want the most probable option. Top-k is good for when you want to inject a bit of creativity (randomness) into the results. This library supports both. I hope to add more sampling options in the future.

Now that we have a list of output tokens, we can convert them back to a string:

```js
// Convert output tokens to a string
const french = tokenizer.decode(outputTokenIds, true);
console.log(french); // "Bonjour monde!"
```

The output is "Bonjour monde!" which makes sense given our input of "Hello World".

That's it! In about 5 lines of code we executed a neural translation algorithm that ran completely in the browser.

Now, let's talk one last detail to make networks in the browser run *well*.


## Optimizing Models for the Browser



## Conclusion

