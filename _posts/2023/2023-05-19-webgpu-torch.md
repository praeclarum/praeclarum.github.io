---
layout: post
title:  "How I Re-implemented PyTorch for WebGPU"
thumbnail: "/images/2023/webgpu-torch.jpg"
tags: article
---

**TL;DR** I've been working on a WebGPU optimized inference and autograd library called [webgpu-torch](https://github.com/praeclarum/webgpu-torch) with an API that matches PyTorch. The goal is to run neural networks at CUDA speeds in the browser. Many kernels have been implemented and its design is easily extensible. It's [available on NPM now](https://www.npmjs.com/package/webgpu-torch) and works in both the browser and Node.js!

## Neural Networks in the Browser

[Nine months ago](https://github.com/praeclarum/transformers-js), I got huggingface transformers (Large Language Models like GPT but a wee bit smaller) working in the browsers thanks to the ONNX web runtime and some painfully hand-coded tokenizers.

It's quite liberating running these nets in the browser since the web is best software distribution platform ever created. You can just send someone a link and they can run your code. No need to install anything. No need to worry about what OS they're running. No need to worry about what hardware they have. It's all just there.

The only problem is that ONNX is a wee bit, shall we say, slow.

Thankfully, WebGPU has arrived in browsers and we can now properly access the GPU to write optimized kernels for neural network operations. This is a huge deal. It means we can now run neural networks in the browser at speeds comparable to NVIDIA/CUDA.

Someone just needs to, you know, do the hard work of implementing all those operations for the GPU.

Well that's what I'm very pleased to announce I've been working on for the past few months. I've been re-implementing PyTorch in TypeScript for WebGPU.

## What is a PyTorch?

PyTorch is a wrapper over the torch runtime (which I first used with Lua) for performing neural network operations. It's a very popular library for doing AI work and seems to have won the arms race for now.

The library is broken up into parts:

1. An optimized (for GPU) math library supporting element-wise operations, matrix multiplication, convolutions, reductions, etc. over tensors.

2. An automatic differentiation library (autograd) that is just a lot of bookkeeping to keep track of the operations performed on tensors so that gradients can be calculated.

3. A neural network library that is just a bunch of layers that can be composed together to form a neural network.

Doesn't sound so hard to re-implement right? And so I did.

## What is a WebGPU?

WebGPU is the new standard for accessing GPUs from the browser. It supports generic compute shaders and is designed to be a low level API that can be used to build higher level libraries. The compute shaders are able to break work up into a 3D grid and, so long as you can reformulate your code to take advantage of that 3D grid, you can benefit from dedicated hardware doing the computations.

This is perfect for the web since JavaScript is single threaded and not optimized for doing heavy computation. The GPU is a perfect fit for this since it's designed to do heavy computation in parallel.

## Writing Optimized WebGPU Kernels

PyTorch is very mature now and supports a huge variety of operations. It's also very well optimized for CUDA and CUDNN (NVIDIA's compute libraries). So how do you go about re-implementing all of those for WebGPU?

Well, you start with the basics. You implement the basic operations like element-wise operations, matrix multiplication, convolutions, reductions, etc. But there is a tremendous amount of similarity between these operations.

For example, element-wise multiplication and addition only vary by the operator used in the inner loop. The trick is to optimize the memory layout and kernels of those operations so they are fast. They need to adapt to big and small GPUs and they need to adapt to big and small workloads.

This is a perfect scenario to take advantage of code generation. I wrote a code generator that takes a template and generates the optimized kernels for each operation. The code generator is written in TypeScript and generates WebGPU compute shader code. This means that the generated code can be heavily optimized for the given scenario and those optimizations can be shared between operations.

For example, here is how I define the `ReLU` operation (from `op_table.ts`):

```typescript
{
    name: "relu",
    nnName: "ReLU",
    nnOp: true,
    type: "unary",
    forward: "output = max(input, 0.0)",
    backward: "inputGrad = input > 0.0 ? outputGrad : 0.0",
}
```

In this template I define both the forward computation `max(input, 0.0)` and the backward computation `input > 0.0 ? outputGrad : 0.0`. The code generator then generates the optimized kernels for both the forward and backward passes based on the size of your GPU (the size of compute workgroups) and the shape of tensors (in addition to the memory layouts of the tensors).

Keeping the template short and simple gives me flexibility to optimize the kernels as needed while preserving the core logic. For example, different kernels can be emitted for contiguous memory tensors vs strided memory tensors. For operations like reductions, 1D, 2D, 3D, and xD kernels can be emitted to take advantage of the 3D workgroup grid.

At first I designed the template system to help me save some typing, but I quickly realized its power and now I use it for all operations.

## Debugging WebGPU Kernels

Another huge benefit came from the fact that I was generating the kernels. I could generate the kernels to not only emit WebGPU code, but also JavaScript code. The core logic gets wrapped in another function that can be called from JavaScript. This means that I can run the same code in JavaScript and WebGPU and compare the results. Even better, I can debug kernels in JavaScript and then execute them on WebGPU.

The JavaScript CPU kernels are terribly slow, but they're not supposed to be fast. They instead provide a convenient playground for debugging and testing kernels.

This also means that my WebGPU library can also run just fine in Node.js, without WebGPU, whatever. Isn't it great when architectural decisions keep paying off?

## Testing

The worst part of using a new neural network library is when it doesn't give the exact same results as previous libraries you've used. One of my biggest frustrations with the WebGL ONNX backend is the fact that it gives very inaccurate results compared to PyTorch. I didn't want that. I want full fidelity. I want to make sure all my WebGPU kernels match the results of PyTorch operations.

To that end, I have built a test harness that first runs code snippets in PyTorch to record results, then runs the same code snippets in my library and compares the results. If they don't match, it throws an error.

This has produced a silly but fun web page to go visit. If you go to [https://praeclarum.org/webgpu-torch/tests/](https://praeclarum.org/webgpu-torch/tests/) you will see a huge set of tests running to verify all the supported operations. It's a great way to see what operations are supported and what the results are.

## Goals

I like to train imaging networks and to that end my goal is to get Stable Diffusion and similar nets running under this library. Once that's accomplished I will focus on the many huggingface transformer networks. I'm hoping to get all of them running in the browser at CUDA speeds.

I have a set of TODOs in the README of the project. If you're interested in helping out, please take a look!
