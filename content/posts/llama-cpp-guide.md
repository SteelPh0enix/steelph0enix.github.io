+++
title = "llama.cpp guide - Hosting local LLMs from scratch"
date = "2024-10-28"
author = "SteelPh0enix"
authorTwitter = "steel_ph0enix"
tags = ["llama.cpp", "llm", "ai", "guide"]
keywords = ["llama.cpp", "llama", "llm", "ai", "building", "running", "guide", "inference"]
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

However, despite ChatGPT not dissapointing me, i was still skeptical.
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
You can even run [LLMs on RaspberryPi's](https://www.reddit.com/r/raspberry_pi/comments/1ati2ki/how_to_run_a_large_language_model_llm_on_a/) at this point (with llama.cpp too!)
Of course, the performance will be *abysmal* if you don't run the LLM with a proper backend on a decent hardware, but the bar is currently not very high.

If you came here with intention of finding some piece of software that will allow you to easily run popular models on most modern hardware - grab LM Studio and go play with it.
It fits this description very well, just make sure to use appropriate back-end for your GPU/CPU for optimal performance.

If you want to know more about llama.cpp, which LM Studio uses under the hood, and/or learn how to build and use it directly - have a nice read!
