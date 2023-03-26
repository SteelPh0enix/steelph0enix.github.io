+++
title = "C/C++ setup guide, refreshed - now with Meson!"
date = "2023-03-26"
author = "SteelPh0enix"
authorTwitter = "steel_ph0enix" #do not include @
cover = ""
tags = ["c", "c++", "cpp", "setup", "guide", "meson"]
keywords = ["c", "c++", "cpp", "setup", "guide", "meson"]
description = "How to setup a C++ environment from scratch and create Meson project"
showFullContent = false
readingTime = true
hideComments = false
+++

## Introduction

Few years ago, i've made [a guide on creating CMake project and integrating it with VSCode](/posts/vscode-cpp-setup/). Back then i knew a few build system for C and C++, but they were mostly niche (like premake), bound to specific environment (like Visual Studio build system), nasty to write manually (Make), and so on - they had issues that made CMake relatively better than them in most scenarios. Recently, my colleague recommended me Meson, and after tinkering with it for a while i found it to be great replacement for CMake, with very reasonable support and even CMake compatibility that allowed for using CMake projects without having to port whole build script to Meson. Unfortunately, Meson support is not as great as CMake's, but in my opinion it's not very problematic as it's relatively easy to use it from anywhere without having to do much manual configuration. So, i'm writing this guide for anyone who's looking for a reasonable replacement for CMake that does not require you to sell your soul to C++ devil in order to use 3rd party libraries, while also giving you relatively easy way of defining C and C++ projects.

### Why Meson and how is it different from CMake?

The main difference between Meson and CMake that i've felt the most is the language syntax. Both Meson and CMake uses their own scripting language, but CMake's is definitely more complex and harder to master. Meson uses relatively simple DSL. If you want more comparisons to other popular build systems, you can find them in Meson documentation, [here](https://mesonbuild.com/Comparisons.html). There's also performance comparison available [here](https://mesonbuild.com/Simple-comparison.html), but long story short - Meson is usually at least as fast as other popular build systems.

The other thing that makes Meson great is the support for 3rd party libraries via [WrapDB](https://mesonbuild.com/Wrapdb-projects.html). CMake also got one, via [FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html), but since Meson is a bit more user-friendly for my taste, it's great that i don't have to manually port the project to Meson if the developers are not supporting it.
