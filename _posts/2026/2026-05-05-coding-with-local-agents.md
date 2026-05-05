---
layout: post
title: "Coding with Local Agents on an RTX 3090"
thumbnail: "/images/2026/local_agents_thumb.jpg"
tags: article
---

**TL;DR** Running coding agents on local machines has never been easier. This article gives easy setup instructions for running Qwen 3.6 27B on an RTX 3090 in Linux. I then show how to use the model in VS Code using the LLM Gateway extension. By the end of this guide, you'll be free of service providers and able to run a variety of OSS models.

## Overview

There are roughly two steps to running a local coding agent:

1. Get the model up and running serving the standard chat API.
2. Connect the model to your coding environment (e.g., VS Code).

There are hundreds of different OSS models, and hundreds of different model servers to choose from. You have, frankly, an overwhelming number of options to fulfill step #1. That said, if you're looking to run these models on consumer-grade hardware, you will be looking at models in the 7B-31B parameter range. Here is one site, of many, that tries to rank these beasts: [Artificial Analysis](https://artificialanalysis.ai/models/open-source/small#intelligence)

For this guide, I will focus on **Qwen 3.6 27B** from Alibaba since it works well-enough. But **Gemma 4 31B** from Google is a champ and is worth also looking at.

There is a wonderful arms race happening with model servers right now too. A model server is a giant math library, optimized into oblivion, that deigns to run an HTTP server so it can service requests. But it also has one more crucial component: a caching layer that keeps as much of chat conversations in GPU memory as possible in order to minimize latency and compute time - the KV cache.

For this guide, I will focus on [**llama.cpp**](https://llama-cpp.com) since it is pretty popular, easy to use, and has good GPU support. But there are a number of other servers that are worth looking at, including [**vLLM**](https://vllm.ai), [**Ollama**](https://ollama.com), [**MLX-LM**](https://github.com/ml-explore/mlx-lm), [**MTPLX**](https://mtplx.com), and on and on.

## Download the Model

This is both the easy part, and the hard part. Easy, because all you have to do is go to Hugging Face and download any of the thousands of models available. It's hard because there are *so many models*! There's model families, model sizes, model fine tunes, model quantizations, model formats. Oh my!

Most inference engines (like llama.cpp) support a specific set of model formats, so that will narrow down your options. For llama.cpp, the supported format is GGUF, so you'll want to look for [models in that format](https://huggingface.co/models?search=gguf). For MLX models (to run on Apple Silicon), you'll look in the [mlx-community](https://huggingface.co/mlx-community).

You'll now need to pick a quantization size. Quantization is a compression method for model weights. If we took a 27 billion parameter model with 32-bit floating point weights, it would be 27B * 32 bits = 108 GB in size. Unless you have a datacenter handy, you won't be running that. Instead, you'll choose, say a 4-bit quantized model. This will compress the weights down to 27B * 4 bits = 13.5 GB, which is much more manageable for consumer hardware. The tradeoff is that quantization can reduce the model's performance and accuracy, but it's often a necessary compromise.

Now the RTX 3090 has 24 GB of VRAM so you might be tempted to pick a higher-bit quantization, but you have to keep in mind that the *context* and the *output* also have to fit in GPU memory. If you want long contexts and long outputs, you might have to go with a lower-bit quantization to ensure everything fits.

The `Q4_K_M` quantization format is a good compromise for a 27B model and a 24 GB GPU. So I'm going to download the `Qwen 3.6 27B Q4_K_M` model from Hugging Face:

```bash
wget "https://huggingface.co/unsloth/Qwen3.6-27B-GGUF/resolve/main/Qwen3.6-27B-Q4_K_M.gguf?download=true"
```

(`wget` is a little dumb, so you'll need to rename the file after downloading it since it doesn't handle the `?download=true` part of the URL very well.)

## Build llama.cpp

You can download prebuilt libraries of llama.cpp but if you want to ensure its optimized for your machine and hardware, you'll want to build it yourself. Thankfully, it's pretty easy to do:

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release -j 8
```

Aside from the nastiness of having to use CMake, building software doesn't get much easier than this.

I passed the `-DGGML_CUDA=ON` flag to ensure that I get NVIDIA CUDA support, which is crucial for running these large models on consumer-grade hardware. If you're on an M-series Mac, you would want to pass `-DGGML_METAL=ON` instead to get support for Apple's Metal API.

If all goes well, you will have a nice, shiny `build/bin/llama-server` executable that you can use to serve your model.

## Run the Server

You will want to run the server with a delicious soup of command line arguments. Something like this:

```bash
./build/bin/llama-server -m ~/Downloads/Qwen3.6-27B-Q4_K_M.gguf --host 0.0.0.0 -ngl 99 -c 262144 -fa on --cache-type-k q4_0 --cache-type-v q4_0
```

Let's deconstruct that soup:

| Argument | Description |
| --- | --- |
| `-m` | The path to the model file you downloaded. |
| `--host 0.0.0.0` | This tells the server to listen on all network interfaces, which is necessary if you want to connect to it from another machine (e.g., your dev machine). |
| `-ngl 99` | This sets the number of GPU layers to use. Setting this to 99 tells the server to use as many GPU layers as possible, which will maximize performance. |
| `-c 262144` | This sets the context size to 262,144 tokens, which is the maximum context size for this model. You can adjust this based on your needs and GPU memory constraints. |
| `-fa on` | This enables the "faster auto-regressive decoding" feature, which can improve performance. |
| `--cache-type-k q4_0 --cache-type-v q4_0` | This sets the quantization type for the KV cache to `q4_0`, which is a good choice for performance and memory efficiency. |

Notice how we are quantizing the KV cache (context and outputs) as well. This is a crucial step for ensuring that the model runs efficiently on consumer-grade hardware, as the KV cache can consume a significant amount of GPU memory.

You'll be greeted with typical programmer excretions:

```text
ggml_cuda_init: found 1 CUDA devices (Total VRAM: 24159 MiB):
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes, VRAM: 24159 MiB
main: n_parallel is set to auto, using n_parallel = 4 and kv_unified = true
build_info: b9026-a817a22bc
system_info: n_threads = 6 (n_threads_batch = 6) / 12 | CUDA : ARCHS = 860 | USE_GRAPHS = 1 | PEER_MAX_BATCH_SIZE = 128 | CPU : SSE3 = 1 | SSSE3 = 1 | AVX = 1 | AVX2 = 1 | F16C = 1 | FMA = 1 | BMI2 = 1 | LLAMAFILE = 1 | OPENMP = 1 | REPACK = 1 | 
Running without SSL
init: using 11 threads for HTTP server
start: binding port with default address family
main: loading model
srv    load_model: loading model '/home/fak/Downloads/Qwen3.6-27B-Q4_K_M.gguf'
common_init_result: fitting params to device memory, for bugs during this step try to reproduce them with -fit off, or provide --verbose logs if the bug only occurs with -fit on
common_params_fit_impl: getting device memory data for initial parameters:
common_memory_breakdown_print: | memory breakdown [MiB] | total    free     self   model   context   compute    unaccounted |
common_memory_breakdown_print: |   - CUDA0 (RTX 3090)   | 24159 = 23257 + (21388 = 15345 +    5206 +     836) +      -20486 |
common_memory_breakdown_print: |   - Host               |                   1214 =   682 +       0 +     532                |
common_params_fit_impl: projected to use 21388 MiB of device memory vs. 23257 MiB of free device memory
common_params_fit_impl: will leave 1868 >= 1024 MiB of free device memory, no changes needed
common_fit_params: successfully fit params to free device memory
common_fit_params: fitting params to free memory took 0.66 seconds
llama_model_loader: loaded meta data with 51 key-value pairs and 851 tensors from /home/fak/Downloads/Qwen3.6-27B-Q4_K_M.gguf (version GGUF V3 (latest))
```

Congratulations. You're now an AI service provider. I recommend getting some seed capital and start selling access to your model to the highest bidder.

But before you do that...

## Install LLM Gateway in VS Code

I rock VS Code for all my coding needs, and I want to be able to use my local model in its AI agent chat window thingy. To do that, I need to install an extension that connects VS Code to the standard chat API. (Why VS Code doesn't support the API standard that literally *every* LLM server provides is beyond me.)

ANYWAY, I like the [LLM Gateway extension](https://marketplace.visualstudio.com/items?itemName=AndrewButson.github-copilot-llm-gateway) by Andrew Butson.

1. Install that extension. 
2. Open the "GitHub Copilot LLM Gateway: Configure Server" UI from the command palette and enter the URL for your server (e.g., `http://my-awesome-server.local:8080`).
3. Test the connection with the "GitHub Copilot LLM Gateway: Test Server Connection" command. It should say "Found 1 model(s)" if everything is working. (If it's not working, email James Montemagno and ask him for help.)
4. Open the "Chat: Manage Language Models" UI from the command palette. You should see your model listed but it will probably be grayed out for some reason. Click it, click the eye ball (gross!), and it should now be active and ready to use in the chat window.
5. Open the chat window, and click the model selector. Choose "Other Models", scroll, and scroll, looking for your model. It's there somewhere. I promise. You might doubt it, but have faith. When in doubt, keep scrolling. You can do it. You found it! Click it, and now you can use your local model in the chat window!


## Is it Worth It?

What does an RTX machine cost these days?

| Component | Price (USD) |
| --- | --- |
| RTX 3090 | $1500 |
| CPU | $300 |
| 64 GB RAM | $700 (what has the world come to?) |
| HDD | $200 |
| PSU | $150 |
| Case | $100 |
| **Total** | **$2,950** |

So for about $3,000 you can have your very own local coding agent. That's a pretty hefty price tag, but it's also a one-time cost.

In a typical day, I burn through about 50,000,000 tokens. 500,000 output tokens, 1,750,000 input tokens, and the rest are cache hits. At 40 tok/second (typical for my RTX), my compute day is about `(500,000 + 1,750,000) / 40 = 56,250 seconds`, which is about **15.6 hours of compute time per day**. Ugh.

Right now, you can use DeepSeek for $3.48 per 1,000,000 output tokens, $1.74 for inputs, and $0.0145 for cache hits. So my daily cost would be `(500,000 / 1,000,000) * 3.48 + (1,750,000 / 1,000,000) * 1.74 + (47,750,000 / 1,000,000) * 0.0145 = $5.48` per day. That's about $1,400 per year (five day work weeks). So in about 2 years, I would recoup the cost of running my own local agent. Hmmm...

So you might not want to run out and buy your own server. But, if you do have an over-provisioned gaming rig, well you might as well put it to use doing something useful. ;-)

## Conclusion

Since 2017 I have been advocating running local models. I'm amazed that it's now possible to run 27B parameter variants on consumer hardware. (In my mind, 7B is still tremendous.) These are real models, able to write good code, in a fully agentic harness. Amazing.

While the up front hardware cost, the noise of fans, and the slower response rates are not ideal and don't make this an easy win, I have a different perspective. AI coding has changed how I work. Permanently. I do not want to go back to writing every line of code by hand, it seems absurd now. But I also don't like being at the mercy of large cloud providers. Having the ability to run my own local agent, even with its limitations, is a huge win for me. I know, even with no internet connection, I can still do what I love: code.

**Colophon:** Written by hand. Proofread and edited by Qwen 3.6 27B running on an RTX 3090.
