+++
title = "llama.cpp guide - Running LLMs locally, on any hardware, from scratch"
date = "2024-10-28"
author = "SteelPh0enix"
authorTwitter = "steel_ph0enix"
tags = ["llama.cpp", "llm", "ai", "guide"]
keywords = ["llama.cpp", "llama", "cpp", "llm", "ai", "building", "running", "guide", "inference", "local", "scratch", "hardware"]
description = "Psst, kid, want some cheap LLMs?"
showFullContent = false
draft = true
+++

## so, i started playing with LLMs...

...and it's pretty fun.
I was very skeptical about the AI/LLM "boom" back when it started.
I thought, like many other people, that they are just mostly making stuff up, and generating uncanny-valley-tier nonsense.
Boy, was i wrong.
I've used ChatGPT once or twice, to test the waters - it made a pretty good first impression, despite hallucinating a bit.
That was back when GPT3.5 was the top model. We came a pretty long way since then.

However, despite ChatGPT not disappointing me, i was still skeptical.
Everything i've wrote, and every piece of response was fully available to OpenAI, or whatever other provider i'd want to use.
This is not a big deal, but it tickles me in a wrong way, and also means i can't use LLMs for any work-related non-open-source stuff.
Also, ChatGPT is free only to a some degree - if i'd want to go full-in on AI, i'd probably have to start paying.
Which, obviously, i'd rather avoid.

At some point i started looking at open-source models.
I had no idea how to use them, but the moment i saw the sizes of "small" models, like Llama 2 7B, i've realized that my RTX 2070 Super with mere 8GB of VRAM would probably have issues running them (i was wrong on that too!), and running them on CPU would probably yield very bad performance.
And then, i've bought a new GPU - RX 7900 XT, with 20GB of VRAM, which is definitely more than enough to run small-to-medium LLMs.
Yay!

Now my issue was finding some software that could run an LLM on that GPU.
CUDA was the most popular back-end - but that's for NVidia GPUs, not AMD.
After doing a bit of research, i've found out about ROCm and found [LM Studio](https://lmstudio.ai/).
And this was exactly what i was looking for - at least for the time being.
Great UI, easy access to many models, and the quantization - that was the thing that absolutely sold me into self-hosting LLMs.
Existence of quantization made me realize that you don't need powerful hardware for running LLMs!
You can even run [LLMs on RaspberryPi's](https://www.reddit.com/r/raspberry_pi/comments/1ati2ki/how_to_run_a_large_language_model_llm_on_a/) at this point (with `llama.cpp` too!)
Of course, the performance will be *abysmal* if you don't run the LLM with a proper backend on a decent hardware, but the bar is currently not very high.

If you came here with intention of finding some piece of software that will allow you to **easily run popular models on most modern hardware for non-commercial purposes** - grab [LM Studio](https://lmstudio.ai/), read the [next section](#but-first---some-disclaimers-for-expectation-management) of this post, and go play with it.
It fits this description very well, just make sure to use appropriate back-end for your GPU/CPU for optimal performance.

However, if you:

- Want to learn more about `llama.cpp` (which LM Studio uses as a back-end), and LLMs in general
- Want to use LLMs for commercial purposes ([LM Studio's terms](https://lmstudio.ai/terms) forbid that)
- Want to run LLMs on exotic hardware (LM Studio provides only the most popular backends)
- Don't like closed-source software (which LM Studio, unfortunately, is) and/or don't trust anything you don't build yourself
- Want to have access to latest features and models as soon as possible

you should find the rest of this post pretty useful!

## but first - some disclaimers for expectation management

Before i proceed, i want to make some stuff clear.
This "FAQ" answers some questions i'd like to know answers to before getting into self-hosted LLMs.

### Do I need RTX 2070 Super/RX 7900 XT ot similar mid/high-end GPU to do what you did here?

No, you don't.
I'll elaborate later, but you can run LLMs with no GPU at all.
As long as you have reasonably modern hardware (by that i mean *at least* a decent CPU with AVX support) - you're *compatible*.
**But remember - your performance may vary.**

### What performance can I expect?

This is a very hard question to answer directly.
The speed of text generation depends from multiple factors, but primarily

- tensor/matrix operation performance on your hardware
- memory bandwidth
- model size

I'll explain this with more details later, but you can generally get reasonable performance from the LLM by picking model small enough for your hardware.
If you intend to use GPU, and it has enough memory for a model with it's context - expect real-time text generation.
In case you want to use both GPU and CPU, or only CPU - you may need to go into very small models to get real-time generation, but it's certainly possible.

### What quality of responses can I expect?

That heavily depends on your usage and chosen model.
I can't answer that question directly, you'll have to play around and find out yourself.
A rule of thumb is "larger the model, better the response" - consider the fact that the size of SOTA (state-of-the-art) LLMs, like GPT-4 or Claude, are usually measured in hundreds of billions of parameters.
Unless you have multiple GPUs or unreasonable amount of RAM and patience - you'll be most likely restricted to models with less than 20 billion parameters.
From my experience, 7B models are pretty good for generic purposes and programming - and they are not *very* far from SOTA models like GPT-4o or Claude in terms of raw quality of generated responses, but the difference can be noticeable at times.
Keep in mind that the choice of a model is only a part of the problem - providing proper context and system prompt, or fine-tuning LLMs can do wonders.

### Can i replace ChatGPT/Claude/\[insert online LLM provider\] with that?

Probably yes.
`llama.cpp` provides OpenAI-compatible server.
As long as your tools communicate with LLMs via OpenAI API, and you are able to set custom endpoint, you will be able to use self-hosted LLM with them.

## getting the basics

Let's start by grabbing a copy of [`llama.cpp` source code](https://github.com/ggerganov/llama.cpp).

```sh
git clone git@github.com:ggerganov/llama.cpp.git
```

If you are very lazy, and you have supported hardware, you can just grab a [release build](https://github.com/ggerganov/llama.cpp/releases) and ignore the "building llama.cpp" section of this post.
If you don't know which version to pick - continue reading.

After cloning the repo (or downloading a zip, if you prefer that), let's see what's inside.
Directly in the repository root, there are few Python scripts for converting models to different formats.
We'll get back to them.
The most interesting subdirectory for us is `examples/`.
I'm gonna cover just few of them, because there's definitely too many for a single blog post (and i don't even know what half of them does at this point).
Those few include a CLI interface and OpenAI-compatible server, which i primarily use for all my LLM needs.
Other than `examples/`, we will want to look at the `docs/` for latest build guide, because whatever i write here will inevitably become outdated at some point.

If you really want to dig into the implementation, you may want to look at `common/`, `src/`, and `include/` contents.
The names are pretty self-explanatory.
`ggml/` directory contains the implementation of GGML tensor library, which is used as a backend for all the tensor math.
`gguf-py/` is a Python package for managing binary files in GGUF (*GGML Universal File*) format, and `requirements/` contains Python requirements for example and utility scripts.

## building the llama

In `docs/build.md`, you'll find detailed build instructions for all the supported platforms.
The most basic build is for the generic CPU support.
All we need now is a working C/C++ toolchain - preferably GCC or Clang.
If you're using Linux, you should be good to go, unless you somehow don't have GCC installed (try `gcc -v` if you're not sure).
If you're using Windows and you don't have your preferred toolchain installed, i personally recommend using [MSYS](https://www.msys2.org/), but Visual C++ is also supported - as long as you install the following components (see `docs/build.md` for details): C++-CMake Tools for Windows, Git for Windows, C++-Clang Compiler for Windows, MS-Build Support for LLVM-Toolset (clang).
I'll describe how to use both MSYS and MSVC, because i'm familiar with them.
