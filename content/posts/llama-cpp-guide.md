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

- matrix operations performance on your hardware
- memory bandwidth
- model size

I'll explain this with more details later, but you can generally get reasonable performance from the LLM by picking model small enough for your hardware.
If you intend to use GPU, and it has enough memory for a model with it's context - expect real-time text generation.
In case you want to use both GPU and CPU, or only CPU - you should expect much lower performance, but real-time text generation is possible with small models.

### What quality of responses can I expect?

That heavily depends on your usage and chosen model.
I can't answer that question directly, you'll have to play around and find out yourself.
A rule of thumb is "larger the model, better the response" - consider the fact that size of SOTA (state-of-the-art) models, like GPT-4 or Claude, is usually measured in hundreds of billions of parameters.
Unless you have multiple GPUs or unreasonable amount of RAM and patience - you'll most likely be restricted to models with less than 20 billion parameters.
From my experience, 7-8B models are pretty good for generic purposes and programming - and they are not *very* far from SOTA models like GPT-4o or Claude in terms of raw quality of generated responses, but the difference is definitely noticeable.
Keep in mind that the choice of a model is only a part of the problem - providing proper context and system prompt, or fine-tuning LLMs can do wonders.

### Can i replace ChatGPT/Claude/\[insert online LLM provider\] with that?

Maybe. In theory - yes, but in practice - it depends on your tools.
`llama.cpp` provides OpenAI-compatible server.
As long as your tools communicate with LLMs via OpenAI API, and you are able to set custom endpoint, you will be able to use self-hosted LLM with them.

## prerequisites

- Reasonably modern CPU. If you're rocking any Ryzen, or Intel's 8th gen or newer, you're good to go, but all of this should work on older hardware too.
- Optimally, a GPU. More VRAM, the better. If you have at least 8GB of VRAM, you should be able to run 7-8B models, i'd say that it's reasonable minimum. Vendor doesn't matter, llama.cpp supports NVidia, AMD and Apple GPUs (not sure about Intel, but i think i saw a backend for that - if not, Vulkan should work).
- If you're not using GPU or it doesn't have enough VRAM, you need RAM for the model. As above, at least 8GB of free RAM is recommended, but more is better.

In the guide, i'll assume you're using either Windows or Linux, with Git, [CMake](https://cmake.org/) and - if you have one - your preferred C++ toolchain. If you don't have one yet, don't worry - i'll tell you where to get one from. If you're on Mac, i'll point you to the guides in llama.cpp repository when not sure about something. Linux and Windows users are also encouraged to check them, as the knowledge in this blog post may not stay up-to-date with whatever llama.cpp does in the future.

Some context-specific formatting is used in this post:

> Parts of this post where i'll write about Windows-specific stuff will have this background.
{.windows-bg}

> And parts where i'll write about Linux-specific stuff will have this background.
{.linux-bg}

## getting the basics

Let's start by grabbing a copy of [`llama.cpp` source code](https://github.com/ggerganov/llama.cpp).

```sh
git clone git@github.com:ggerganov/llama.cpp.git
```

If you are very lazy, and you have supported hardware, you can just grab a [release build](https://github.com/ggerganov/llama.cpp/releases) and ignore the "building the llama" section of this post.
If you don't know which version to pick - continue reading.

## building the llama

In [`docs/build.md`](https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md), you'll find detailed build instructions for all the supported platforms.
By default, `llama.cpp` builds with auto-detected CPU support.
We'll talk about GPU support in a moment, first - let's try building it as-is, because it's a good baseline to start with, and it doesn't require any external dependencies.
To do that, we only need a C++ toolchain, [CMake](https://cmake.org/) and [Ninja](https://ninja-build.org/).

> On Windows, you can use [Microsoft Visual C++](https://visualstudio.microsoft.com/downloads/) toolchain, but if you don't have it already installed i recommend using [MSYS](https://www.msys2.org/) to install MinGW instead.
> It's smaller and saves some headaches.
> Follow the guide on the main page to install MinGW for x64 UCRT environment, which you probably should be using.
> CMake and Ninja can be installed via `winget`:
>
> ```ps
> winget install cmake ninja-build.ninja
> ```
>
> However, if you're using MSVC, you should install CMake via Visual Studio installer (`Build Tools` -> `C++ CMake tools for Windows`) as a package.
{.windows-bg}

> On Linux, GCC is recommended, but you should be able to use Clang if you'd prefer, by setting `CMAKE_C_COMPILER=clang` and `CMAKE_CXX_COMPILER=clang++` variables.
> If you don't have any toolchain installed (try `gcc --version` in terminal), check the documentation of your distribution (or Google) for installation instructions.
{.linux-bg}

After you install the toolchain, CMake and Ninja (if you haven't done that already), open terminal in directory with cloned `llama.cpp` repository.
You may also want to install `ccache` if you intend to rebuild `llama.cpp` often, it will save you some time, but it's not required.

```sh
cd llama.cpp
```

Now we'll use CMake to generate build files, build the project, and install it.
Run the following command to generate build files in `build/` subdirectory:

```sh
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/your/install/dir -DLLAMA_BUILD_TESTS=OFF -DLLAMA_BUILD_EXAMPLES=ON -DLLAMA_BUILD_SERVER=ON -DLLAMA_STANDALONE=ON
```

There's a lot of CMake variables being defined, which we could ignore and let llama.cpp use it's defaults, but we won't:

- `CMAKE_BUILD_TYPE` is set to release for obvious reasons - we want maximum performance.
- [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html) is where the `llama.cpp` binaries and python scripts will go.
  - On Linux, default directory is `/usr/local`, you can ignore this variable if that's fine with you, but you'll need super-user permissions to install the binaries. If you don't have them, change it to point somewhere in your user directory.
  {.linux-bg-padded}
  - On Windows, default directory is `c:/Program Files/llama.cpp` - as above, you'll need admin privileges to install it, and you'll have to add the `bin/` subdirectory to your `PATH` to make llama.cpp binaries accessible system-wide. I prefer installing llama.cpp in `$env:LOCALAPPDATA/llama.cpp` (`C:/Users/me/AppData/Local/llama.cpp`), as it doesn't require admin privileges.
  {.windows-bg-padded}

Now, let's build the project.
Replace `X` with amount of cores your CPU has for faster compilation.
In theory, Ninja should automatically use all available cores, but i still prefer passing this argument manually.

```sh
cmake --build build --config Release -j X
```
