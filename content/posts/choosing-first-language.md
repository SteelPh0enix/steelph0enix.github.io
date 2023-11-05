+++
title = "When should you choose C++ as your starting language?"
date = "2023-11-05"
author = "SteelPh0enix"
authorTwitter = "steel_ph0enix" #do not include @
cover = ""
tags = ["random-thoughts", "guide"]
keywords = ["pick", "language", "first", "c", "c++"]
description = "Short answer: probably never."
showFullContent = false
lastmod = "2023-11-05"
+++

When i'm lurking through the internet, i'm often seeing posts asking about "which language should i pick as a beginner???". As someone who struggled a lot with this choice, and ultimately picked C++ (for reasons that made no real sense - but, of course, i didn't knew that back then), i think i can say a few words about this specific choice and how it can affect the learning process of an individual.

## It's a trap!

TL;DR of this post - it's usually not a good idea to pick C++ as your starting language. After working with C++ for a long time, and tasting many different programming languages, i feel like C++ is a convoluted mess taped together using a subpar-quality duct tape, somehow still holding on, maybe even going in a relatively good direction with recent changes, but certainly not good enough for a beginner to learn the programming principles on at it's current state.

## Why would you even want to do that?

Excellent question! In most cases, i hear these specific arguments, trying very hard to justify picking C++ as a starter:

* **It's very fast!** - very common misconception. **Languages are not inherently fast nor slow**. Sure, some languages can be *parsed* or *interpreted*, faster than others, but it does not imply that the **program** written in language A will *always* be faster than program written in language B, or vice versa. Surely, that can be a thing - we absolutely can write a program in C++ that will be faster than equivalent program in Python or Java, but **it works both ways!** One of my favorite quotes about the C++ "speed" is from [this site](https://www.simplilearn.com/tutorials/cpp-tutorial/cpp-vs-python) that showed up as one of the top searches in Google - *C++ is faster than Python because it is statically typed, which leads to a faster compilation of code*, i roll my eyes (because python is well-known for it's long compilation times... oh, wait, it's the other way around). **When you reach certain level of complexity in your program, the choice of language starts to matter less, as the choice of algorithms and data structures starts to matter more**. Also - it's usually good thing to ask yourself *why program written in language A may be faster/slower than equivalent program written in language B*, but that's not really something that a newbie should care about. Generally speaking, you should NOT care about "language performance" as a beginner, as it's one of the last things you'll have to worry about when learning programming.
* **Learning C++ teaches you low-level concepts, like pointers and manual memory management!** - It can, but there's a biiiiiig **but**. First - you don't have to know these low-level concepts to write software and learn programming. Every day thousands of programmers write perfectly fine and working code without even knowing what pointer is, or how to manually manage the memory. It's not something that you absolutely must know in order to write working programs.
