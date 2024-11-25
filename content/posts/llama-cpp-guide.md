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
The speed of text generation depends on multiple factors, but primarily

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

- Reasonably modern CPU.
  If you're rocking any Ryzen, or Intel's 8th gen or newer, you're good to go, but all of this should work on older hardware too.
- Optimally, a GPU.
  More VRAM, the better.
  If you have at least 8GB of VRAM, you should be able to run 7-8B models, i'd say that it's reasonable minimum.
  Vendor doesn't matter, llama.cpp supports NVidia, AMD and Apple GPUs (not sure about Intel, but i think i saw a backend for that - if not, Vulkan should work).
- If you're not using GPU or it doesn't have enough VRAM, you need RAM for the model.
  As above, at least 8GB of free RAM is recommended, but more is better.
  Keep in mind that when only GPU is used by llama.cpp, RAM usage is very low.

In the guide, i'll assume you're using either Windows or Linux.
I can't provide any support for Mac users, so they should follow Linux steps and consult the llama.cpp docs wherever possible.

Some context-specific formatting is used in this post:

> Parts of this post where i'll write about Windows-specific stuff will have this background.
> You'll notice they're much longer than Linux ones - Windows is a PITA.
> Linux is preferred. I will still explain everything step-by-step for Windows, but in case of issues - try Linux.
{.windows-bg}

> And parts where i'll write about Linux-specific stuff will have this background.
{.linux-bg}

## building the llama

In [`docs/build.md`](https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md), you'll find detailed build instructions for all the supported platforms.
By default, `llama.cpp` builds with auto-detected CPU support.
We'll talk about GPU support in a moment, first - let's try building it as-is, because it's a good baseline to start with, and it doesn't require any external dependencies.
To do that, we only need a C++ toolchain, [CMake](https://cmake.org/) and [Ninja](https://ninja-build.org/).

> If you are **very** lazy, you can download a release from Github and skip building steps.
> Make sure to download correct version for your hardware/backend.
> If you have troubles picking, i recommend following the build guide anyway - it's simple enough and should explain what you should be looking for.
> Keep in mind that release won't contain Python scripts that we're going to use, so if you'll want to quantize models manually, you'll need to get them from repository.

> On Windows, i recommend using [MSYS](https://www.msys2.org/) to setup the environment for building and using `llama.cpp`.
> [Microsoft Visual C++](https://visualstudio.microsoft.com/downloads/) is supported too, but trust me on that - you'll want to use MSYS instead (it's still a bit of pain in the ass, Linux setup is much simpler).
> Follow the guide on the main page to install MinGW for x64 UCRT environment, which you probably should be using.
> CMake, Ninja and Git can be installed in UCRT MSYS environment like that:
>
> ```sh
> pacman -S git mingw-w64-ucrt-x86_64-cmake mingw-w64-ucrt-x86_64-ninja
> ```
>
> However, if you're using any other toolchain (MSVC, or non-MSYS one), you should install CMake, Git and Ninja via `winget`:
>
> ```powershell
> winget install cmake git.git ninja-build.ninja
> ```
>
> You'll also need Python, which you can get via winget.
> Get the latest available version, at the time of writing this post it's 3.13.
>
> **DO NOT USE PYTHON FROM MSYS, IT WILL NOT WORK PROPERLY DUE TO ISSUES WITH BUILDING `llama.cpp` DEPENDENCY PACKAGES!**
> **We're going to be using MSYS only for *building* `llama.cpp`, nothing more.**
>
> **If you're using MSYS, remember to add it's `/bin` (`C:\msys64\ucrt64\bin` by default) directory to PATH, so Python can use MinGW for building packages.**
> **Check if GCC is available by opening PowerShell/Command line and trying to run `gcc --version`.**
> Also; check if it's *correct* GCC by running `where.exe gcc.exe` and seeing where the first entry points to.
> Reorder your PATH if you'll notice that you're using wrong GCC.
>
> **If you're using MSVC - ignore this disclaimer, it should be "detectable" by default.**
>
> ```powershell
> winget install python.python.3.13
> ```
>
> I recommend installing/upgrading `pip`, `setuptools` and `wheel` packages before continuing.
>
> ```powershell
> python -m pip install --upgrade pip wheel setuptools
> ```
>
{.windows-bg}

> On Linux, GCC is recommended, but you should be able to use Clang if you'd prefer by setting `CMAKE_C_COMPILER=clang` and `CMAKE_CXX_COMPILER=clang++` variables.
> You should have GCC preinstalled (check `gcc --version` in terminal), if not - get latest version for your distribution using your package manager.
> Same applies to CMake, Ninja, Python 3 (with `setuptools`, `wheel` and `pip`) and Git.
{.linux-bg}

Let's start by grabbing a copy of [`llama.cpp` source code](https://github.com/ggerganov/llama.cpp), and moving into it.

> Disclaimer: this guide assumes all commands are ran from user's home directory (`/home/[yourusername]` on Linux, `C:/Users/[yourusername]` on Windows).
> You can use any directory you'd like, just keep in mind that if "starting directory" is not explicitly mentioned, start from home dir/your chosen one.

> **If you're using MSYS**, remember that MSYS home directory is different from Windows home directory. Make sure to use `cd` (without arguments) to move into it after starting MSYS.
{.windows-bg}

```sh
git clone git@github.com:ggerganov/llama.cpp.git
cd llama.cpp
git submodule update --init --recursive
```

Now we'll use CMake to generate build files, build the project, and install it.
Run the following command to generate build files in `build/` subdirectory:

```sh
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/your/install/dir -DLLAMA_BUILD_TESTS=OFF -DLLAMA_BUILD_EXAMPLES=ON -DLLAMA_BUILD_SERVER=ON
```

There's a lot of CMake variables being defined, which we could ignore and let llama.cpp use it's defaults, but we won't:

- `CMAKE_BUILD_TYPE` is set to release for obvious reasons - we want maximum performance.
- [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html) is where the `llama.cpp` binaries and python scripts will go. Replace the value of this variable, or remove it's definition to keep default value.
  - On Windows, default directory is `c:/Program Files/llama.cpp`.
    As above, you'll need admin privileges to install it, and you'll have to add the `bin/` subdirectory to your `PATH` to make llama.cpp binaries accessible system-wide.
    I prefer installing llama.cpp in `$env:LOCALAPPDATA/llama.cpp` (`C:/Users/[yourusername]/AppData/Local/llama.cpp`), as it doesn't require admin privileges.
  {.windows-bg-padded}
  - On Linux, default directory is `/usr/local`.
    You can ignore this variable if that's fine with you, but you'll need superuser permissions to install the binaries there.
    If you don't have them, change it to point somewhere in your user directory and add it's `bin/` subdirectory to `PATH`.
  {.linux-bg-padded}
- `LLAMA_BUILD_TESTS` is set to `OFF` because we don't need tests, it'll make the build a bit quicker.
- `LLAMA_BUILD_EXAMPLES` is `ON` because we're gonna be using them.
- `LLAMA_BUILD_SERVER` - see above. Note: Disabling `LLAMA_BUILD_EXAMPLES` unconditionally disables building the server, both must be `ON`.

Now, let's build the project.
Replace `X` with amount of cores your CPU has for faster compilation.
In theory, Ninja should automatically use all available cores, but i still prefer passing this argument manually.

```sh
cmake --build build --config Release -j X
```

Building should take only a few minutes.
After that, we can install the binaries for easier usage.

```sh
cmake --install build --config Release
```

Now, after going to `CMAKE_INSTALL_PREFIX/bin` directory, we should see a list of executables and Python scripts:

```sh
/c/Users/phoen/llama-build/bin
❯ l
Mode  Size Date Modified Name
-a--- 203k  7 Nov 16:14  convert_hf_to_gguf.py
-a--- 3.9M  7 Nov 16:18  llama-batched-bench.exe
-a--- 3.9M  7 Nov 16:18  llama-batched.exe
-a--- 3.4M  7 Nov 16:18  llama-bench.exe
-a--- 3.9M  7 Nov 16:18  llama-cli.exe
-a--- 3.2M  7 Nov 16:18  llama-convert-llama2c-to-ggml.exe
-a--- 3.9M  7 Nov 16:18  llama-cvector-generator.exe
-a--- 3.9M  7 Nov 16:18  llama-embedding.exe
-a--- 3.9M  7 Nov 16:18  llama-eval-callback.exe
-a--- 3.9M  7 Nov 16:18  llama-export-lora.exe
-a--- 3.0M  7 Nov 16:18  llama-gbnf-validator.exe
-a--- 1.2M  7 Nov 16:18  llama-gguf-hash.exe
-a--- 3.0M  7 Nov 16:18  llama-gguf-split.exe
-a--- 1.1M  7 Nov 16:18  llama-gguf.exe
-a--- 3.9M  7 Nov 16:18  llama-gritlm.exe
-a--- 3.9M  7 Nov 16:18  llama-imatrix.exe
-a--- 3.9M  7 Nov 16:18  llama-infill.exe
-a--- 4.2M  7 Nov 16:18  llama-llava-cli.exe
-a--- 3.9M  7 Nov 16:18  llama-lookahead.exe
-a--- 3.9M  7 Nov 16:18  llama-lookup-create.exe
-a--- 1.2M  7 Nov 16:18  llama-lookup-merge.exe
-a--- 3.9M  7 Nov 16:18  llama-lookup-stats.exe
-a--- 3.9M  7 Nov 16:18  llama-lookup.exe
-a--- 4.1M  7 Nov 16:18  llama-minicpmv-cli.exe
-a--- 3.9M  7 Nov 16:18  llama-parallel.exe
-a--- 3.9M  7 Nov 16:18  llama-passkey.exe
-a--- 4.0M  7 Nov 16:18  llama-perplexity.exe
-a--- 3.0M  7 Nov 16:18  llama-quantize-stats.exe
-a--- 3.2M  7 Nov 16:18  llama-quantize.exe
-a--- 3.9M  7 Nov 16:18  llama-retrieval.exe
-a--- 3.9M  7 Nov 16:18  llama-save-load-state.exe
-a--- 5.0M  7 Nov 16:19  llama-server.exe
-a--- 3.0M  7 Nov 16:18  llama-simple-chat.exe
-a--- 3.0M  7 Nov 16:18  llama-simple.exe
-a--- 3.9M  7 Nov 16:18  llama-speculative.exe
-a--- 3.1M  7 Nov 16:18  llama-tokenize.exe
```

Don't feel overwhelmed by the amount, we're only going to be using few of them.
You should try running one of them to check if the executables have built correctly, for example - try `llama-cli --help`.
We can't do anything meaningful yet, because we lack a single critical component - a model to run.

## getting a model

The main place to look for models is [HuggingFace](https://huggingface.co/).
You can also find datasets and other AI-related stuff there, great site.

We're going to use [`SmolLM2`](https://huggingface.co/collections/HuggingFaceTB/smollm2-6723884218bcda64b34d7db9) here, a model series created by HuggingFace and published fairly recently (1st November 2024).
The reason i've chosen this model is the size - as the name implies, it's *small*.
Largest model from this series has 1.7 billion parameters, which means that it requires approx. 4GB of system memory to run in *raw, unquantized* form (excluding context)!
There are also 360M and 135M variants, which are even smaller and should be easily runnable on RaspberryPi or a smartphone.

There's but one issue - `llama.cpp` cannot run "raw" models directly.
What is usually provided by most LLM creators are original weights in `.safetensors` or similar format.
`llama.cpp` expects models in `.gguf` format.
Fortunately, there is a very simple way of converting original model weights into `.gguf` - `llama.cpp` provides `convert_hf_to_gguf.py` script exactly for this purpose!
Sometimes the creator provides `.gguf` files - for example, two variants of `SmolLM2` are provided by HuggingFace in this format.
This is not a very common practice, but you can also find models in `.gguf` format uploaded there by community.
However, i'll ignore the existence of pre-quantized `.gguf` files here, and focus on quantizing our models by ourselves here, as it'll allow us to experiment and adjust the quantization parameters of our model without having to download it multiple times.

Grab the content of [SmolLM2 1.7B Instruct](https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B-Instruct/tree/main) repository (you can use [360M Instruct](https://huggingface.co/HuggingFaceTB/SmolLM2-360M-Instruct/tree/main) or [135M Instruct](https://huggingface.co/HuggingFaceTB/SmolLM2-135M-Instruct/tree/main) version instead, if you have less than 4GB of free (V)RAM - or use any other model you're already familiar with, if it supports `transformers`), but omit the LFS files - we only need a single one, and we'll download it manually.

> Why *Instruct*, specifically?
> You might have noticed that there are two variants of all those models - *Instruct* and *the other one without a suffix*.
> *Instruct* is trained for chat conversations, base model is only trained for text completion and is usually used as a base for further training.
> This rule applies to most LLMs, but not all, so make sure to read the model's description before using it!

If you're using Bash/ZSH or compatible shell:
{.linux-bg-padded}

*MSYS uses Bash by default, so it applies to it too.*
*From now on, assume that Linux commands work on MSYS too, unless i explicitly say otherwise.*
{.windows-bg-padded}

```sh
GIT_LFS_SKIP_SMUDGE=1 git clone https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B-Instruct
```

If you're using PowerShell:
{.windows-bg-padded}

```sh
$env:GIT_LFS_SKIP_SMUDGE=1
git clone https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B-Instruct
```

If you're using cmd.exe (VS Development Prompt, for example):
{.windows-bg-padded}

```sh
set GIT_LFS_SKIP_SMUDGE=1
git clone https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B-Instruct
```

HuggingFace also supports Git over SSH. You can look up the `git clone` command for every repo here:
![huggingface - where to find git clone button](/img/llama-cpp/clone-hf-button.png)

After cloning the repo, **download the `model.safetensors` file from HuggingFace manually.**
The reason why we used `GIT_LFS_SKIP_SMUDGE` is because there's many other large model files hidden in repo, and we don't need them.
Also, downloading very large files manually is faster, because Git LFS sucks in that regard.

After downloading everything, our local copy of the SmolLM repo should look like this:

```sh
PS D:\LLMs\repos\SmolLM2-1.7B-Instruct> l
Mode  Size Date Modified Name
-a---  806  2 Nov 15:16  all_results.json
-a---  888  2 Nov 15:16  config.json
-a---  602  2 Nov 15:16  eval_results.json
-a---  139  2 Nov 15:16  generation_config.json
-a--- 515k  2 Nov 15:16  merges.txt
-a--- 3.4G  2 Nov 15:34  model.safetensors
d----    -  2 Nov 15:16  onnx
-a---  11k  2 Nov 15:16  README.md
d----    -  2 Nov 15:16  runs
-a---  689  2 Nov 15:16  special_tokens_map.json
-a--- 2.2M  2 Nov 15:16  tokenizer.json
-a--- 3.9k  2 Nov 15:16  tokenizer_config.json
-a---  240  2 Nov 15:16  train_results.json
-a---  89k  2 Nov 15:16  trainer_state.json
-a---  129  2 Nov 15:16  training_args.bin
-a--- 801k  2 Nov 15:16  vocab.json
```

We're gonna (indirectly) use only four of those files:

- `config.json` contains configuration/metadata of our model
- `model.safetensors` contains model weights
- `tokenizer.json` contains tokenizer data (mapping of text tokens to their ID's, and other stuff).
  Sometimes this data is stored in `tokenizer.model` file instead.
- `tokenizer_config.json` contains tokenizer configuration (for example, special tokens and chat template)
{.pre-anti-plag}

i'm leaving this sentence here as anti-plagiarism token.
If you're not currently reading this on my blog, which is @ steelph0enix.github.io, someone probably stolen that article without permission
{.anti-plag}

### converting huggingface model to GGUF {.post-anti-plag}

In order to convert this raw model to something that `llama.cpp` will understand, we'll use aforementioned `convert_hf_to_gguf.py` script that comes with `llama.cpp`.
This script requires some Python libraries, one of which also comes with `llama.cpp`.
For all our Python needs, we're gonna need a virtual environment.
I recommend making it outside of `llama.cpp` repo, for example - in your home directory.

First, we need to make one.

On Linux, run this command (tweak the path if you'd like):
{.linux-bg-padded}

```sh
python -m venv ~/llama-cpp-venv
```

If you're using PowerShell, this is the equivalent:
{.windows-bg-padded}

```powershell
python -m venv $env:USERPROFILE/llama-cpp-venv
```

If you're using cmd.exe, this is the equivalent:
{.windows-bg-padded}

```batch
python -m venv %USERPROFILE%/llama-cpp-venv
```

Then, we need to activate it.

On Linux:
{.linux-bg-padded}

```sh
source ~/llama-cpp-venv/bin/activate
```

With PowerShell:
{.windows-bg-padded}

```powershell
. $env:USERPROFILE/llama-cpp-venv/Scripts/Activate.ps1
```

With cmd.exe:
{.windows-bg-padded}

```cmd
call %USERPROFILE%/llama-cpp-venv/Scripts/activate.bat
```

After that, let's make sure that our virtualenv has all the core packages up-to-date.

```sh
python -m pip install --upgrade pip wheel setuptools
```

Next, we need to install prerequisites for the llama.cpp scripts.
This will be a two-step process.
First, we'll install `llama.cpp` dependencies, and then we'll install `gguf` library from `llama.cpp` repository.
Let's look into `requirements/` directory of our `llama.cpp` repository.
We should see something like this:

```sh
❯ l llama.cpp/requirements
Mode  Size Date Modified Name
-a---  428 11 Nov 13:57  requirements-all.txt
-a---   34 11 Nov 13:57  requirements-compare-llama-bench.txt
-a---  111 11 Nov 13:57  requirements-convert_hf_to_gguf.txt
-a---  111 11 Nov 13:57  requirements-convert_hf_to_gguf_update.txt
-a---   99 11 Nov 13:57  requirements-convert_legacy_llama.txt
-a---   43 11 Nov 13:57  requirements-convert_llama_ggml_to_gguf.txt
-a---   96 11 Nov 13:57  requirements-convert_lora_to_gguf.txt
-a---   48 11 Nov 13:57  requirements-pydantic.txt
-a---   13 11 Nov 13:57  requirements-test-tokenizer-random.txt
```

As we can see, there's a file with deps for our script!
To install dependencies from it, run this command:

```sh
python -m pip install -r llama.cpp/requirements/requirements-convert_hf_to_gguf.txt
```

If `pip` failed during build, make sure you have working C/C++ toolchain in your `PATH`.

> If you're using MSYS, don't. Go back to Windows, install Python via winget and repeat the setup.
> As far as i've tested it, Python deps don't detect the platform correctly on MSYS and try to use wrong build stuff.
> I've warned you at the beginning about it.
{.windows-bg}
