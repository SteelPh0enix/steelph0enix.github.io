+++
title = "Getting Started with Meson - part 1"
date = "2024-01-21"
author = "SteelPh0enix"
authorTwitter = "steel_ph0enix"
tags = ["c", "cpp", "meson", "guide"]
keywords = ["c", "cpp", "c++", "meson", "project", "template", "getting", "started", "guide"]
description = "We're cooking a reasonable Meson template for C/C++ projects"
showFullContent = false
+++

## Intro

One of the things I dislike about C and C++ in particular is lack of standardized build environment, package manager, and all that stuff.
And although CMake is de-facto "standard" over here, I'm not gonna go that way.
You see, I don't really *like* CMake. It's not a *bad* tool, I've used it few times, I've seen some non-trivial projects made with CMake, but I did not like what I've seen.
I cannot really put a finger on it, but I feel that it's maybe a bit overcomplicated, or non-friendly to new users.
And yes, I should **expect** that, considering that I'm trying to make something using C/C++ after all... But I don't have to **accept** that.

So I'm going with [Meson Build system](https://mesonbuild.com/), which was recommended to me by some colleague few years ago.
Since then I've read the docs and also experimented with it a bit, but I haven't made a proper project as of writing this sentence yet.

## Goals

My main goal here is *seemingly* simple: **Make a generic, easy-to-expand project template in Meson for small-to-medium size C/C++ projects**.
I also need some specific features there, that would make my life much simpler.
In the end, I will use that template to write some kind of application and maybe play with the project's structure afterwards, until I'm satisfied with the results.
After that, I have a plan to use this template as a basis for ARM Cortex-M project template, but we'll cross that bridge when we'll get to it.

The first thing that I'm interested in is unit and integration test support.
I don't want to make assumptions about test harnesses though, so I'll just have to make sure that the project's structure is *testable* and Meson can build the tests.
Unit tests have built-in support in Meson, so I'll go with that.
There's also some support for code coverage, and I will surely explore that too.
Integration tests are much more project-dependent, so the only requirements there are that they must be possible to define via Meson (as build targets, for example) and runnable, but running them might be done via external script.
Meson is our *build* system, not necessarily the runner, nor the harness, so I do not expect it to be able to handle generic integration tests by itself.
Some external tooling/scripting may still be required, but everything should eventually be held together by Meson.

The second thing that I wanna see there is support for C/C++ LSP.
My primary choice there is `clangd`, so we'll have to generate `compile_commands.json` using Meson. Fortunately, it does that automatically.

The third thing is support for code checks and code formatting.
External tools can be plugged to Meson via [custom build targets](https://mesonbuild.com/Custom-build-targets.html), and I'll configure at least one code checker and autoformatter like that.
Our project should provide commands to perform project-wide code and formatting check (which is very useful in git prehooks).

And the last, but not the least - documentation.
I want to be able to generate docs for my code.
Probably using Doxygen, as it's relatively easy to set up.
This also includes coverage reports, and I will most likely use `lcov`/`gcov` for that.

>*Did I mention that I've spent most of last year writing Rust code?*
>*Did you know that Cargo provides most of things that I've described here out-of-the-box?*
>*Now you do.*

