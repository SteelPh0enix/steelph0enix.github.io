+++
title = "Making C/C++ project template in Meson - part 1"
date = "2024-01-21"
author = "SteelPh0enix"
authorTwitter = "steel_ph0enix"
tags = ["c", "cpp", "meson", "guide"]
keywords = ["c", "cpp", "c++", "meson", "project", "template", "getting", "started", "guide"]
description = "We're cooking a reasonable Meson template for C/C++ projects"
showFullContent = false
+++

## Intro

One of the things that I dislike about C and C++ in particular is lack of standardized build environment, package manager, and all that stuff.
And although CMake is de-facto "standard" over here, I'm not gonna go that way.
You see, I don't really *like* CMake. It's not a *bad* tool, I've used it few times, I've seen some non-trivial projects made with CMake, but I did not like what I've seen.
I cannot really put a finger on it, but I feel that it's maybe a bit overcomplicated, or non-friendly to new users.
And yes, I should **expect** that, considering that I'm trying to make something using C/C++ after all... But I don't have to **accept** that.

For a very long time I've been missing a proper C/C++ project template.
I was not aware I'm missing one, until i got a hang of the codebase of one of the projects I've been working on in my work, and experienced the true power of scripting and automation.
I've been tinkering with many build systems over the years, but I've never found one that really "clicked" for me.
And since I still cannot lift the curse of C/C++ from my life, it seems the only alternative I have is to pick *something* and force myself through.
And I'm taking You for a ride with me.

So we're going with [Meson Build system](https://mesonbuild.com/), which was recommended to me by a colleague few years ago.
Since then I've read the docs and also experimented with it a bit, but I haven't made a proper project based on it, as of writing this sentence yet.
This is about to change.

I will try to make this series of blog posts as "followable" as possible, so You should be able to reproduce most of my work by yourself, and understand the choices I make. 
Remember that the best tool for the job is sometimes the one you make yourself, not the one you blindly copy&paste without deeper understanding.
~~But please, don't reinvent the wheel if that's not necessary.~~

## Goals

Our main goal here is *seemingly* simple: **Make a generic, compiler/platform-independent, easy-to-expand project template in Meson for small-to-medium size C/C++ projects**.
I also want it to have some specific features that I believe are "must have" in any project.
In the end, we will use that template to write some kind of application and maybe play with the project's structure afterwards, until I'm satisfied with the results.
After that, I have a plan to use this template as a basis for ARM Cortex-M project template, but we'll cross that bridge when we'll get to it.

The first thing that I'm interested in is unit and integration test support.
I don't want to make assumptions about test harnesses though, so we'll just have to make sure that the project's structure is *testable* and Meson can build the tests.
Unit tests have built-in support in Meson, so let's go with that.
There's also some support for code coverage, and we will surely explore that too.
Integration tests are much more project-dependent, so the only requirements there are that they must be possible to define via Meson (as build targets, for example) and runnable, but running them might be done via external script.
Meson is our *build* system, not necessarily the runner, nor the harness, so let's not expect it to be able to handle generic integration tests by itself.
Some external tooling/scripting may still be required, but everything should eventually be held together by Meson.

The second thing that I wanna see there is support for C/C++ LSP.
My primary choice there is `clangd`, so we'll have to generate `compile_commands.json` using Meson. Fortunately, it does that automatically.

The third thing is support for code checks and code formatting.
External tools can be plugged to Meson via [custom build targets](https://mesonbuild.com/Custom-build-targets.html), and we will configure at least one code checker and autoformatter like that.
Our project should provide commands to perform project-wide code and formatting check (which is very useful in git prehooks).

And the last, but not the least - documentation.
I want to be able to generate docs for my code via Meson.
Probably using Doxygen, as it's relatively easy to set up.
This also includes coverage reports, and we will most likely use `lcov`/`gcov` for that.

>*Did I mention that I've spent most of last year writing Rust code?*
>*Did you know that Cargo provides most of the stuff that I've described here out-of-the-box?*
>*Now you do.*

## Preparations

Before we start, let's check our prerequisites, because there's gonna be some.

* Meson - on Linux, should be in your package manager, it's fairly popular piece of software. On Windows, i recommend grabbing an installer from [Github](https://github.com/mesonbuild/meson/releases). Just make sure it's in your `PATH` variable - check if opening a terminal and running `meson --version` returns expected version string. I'm currently running 1.3.1. *Technically Meson is available via `winget`, but their repo currently provides outdated version, so i recommend installing it manually on Windows.*
* Ninja - I'll describe it's purpose in a bit. It should be bundled with Meson on Windows, and should be installed automatically on Linux when installing Meson via package manager. Should be in your `PATH` too. In any case, verify if it's installed correctly by running `ninja --version`. I'm currently running 1.11.1. If you're missing it, you can install it manually [by downloading a zip from Github](https://github.com/ninja-build/ninja/releases) and extracting it into a directory that's in `PATH` variable.
* C/C++ toolchain - In theory, we don't have to choose any specific toolchain, because we're using Meson and Ninja which support the popular ones. **However**, I've mentioned that I want to have coverage report generation, and the only coverage tools that i'm familiar with are `lcov`/`gcov`, and this pretty much forces me to use GCC. If you don't care about code coverage reports, pick your treat. If you somehow got here as a complete newbie and you don't have any C/C++ toolchain installed, either grab GCC from your package manager if you're running Linux (look for package called `build-essentials` or `base-devel` or something like that), or - if you're running Windows - [download latest WinLibs package and add it's `bin` subdirectory to PATH](/posts/vscode-cpp-setup/#cc-toolchain). Or install Microsoft's Visual C++ toolchain via Visual Studio Build Tools (you may need to do that even after installing WinLibs, if you want to use `clang`, but I recommend **not** using MSVC for pure C unless you don't value your sanity). **TL;DR install latest GCC**.
* LLVM tools - mostly `clangd`. If you're running Linux, again - find them in your package manager. If you're running Windows and WinLibs, you already got them. If you don't use WinLibs, install latest LLVM release (`winget install llvm`, should be "recent enough", or get it from [Github](https://github.com/llvm/llvm-project/releases)), it should contain everything we'll need. Again, if manually installed - make sure it's in PATH by running `clangd --version`. I'm currently using 17.0.5.
* Doxygen - we're not gonna be using it any time soon, but let's make sure it's available. Linux - install from repo, Windows - it's already in WinLibs, if not using WinLibs (or you want the latest version), install manually - `winget install doxygen` or grab installer from [here](https://www.doxygen.nl/download.html). Run `doxygen --version` to verify, i'm using 1.10.0. **Note - Winget may not add it to PATH, it's installed in `Program Files/doxygen` by default. Add `bin` subdir to `PATH` manually if that's the case.**

Oh, and there's a small issue of `lcov` not having official Windows support. There are some unofficial releases, but we're not gonna use them - instead, we're gonna lock the coverage report generation feature to Linux and GCC only.
I don't want to completely lock this template to GCC because of that feature, so we will have to separate this part of it appropriately.
