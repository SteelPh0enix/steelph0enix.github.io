+++
title = "Making C/C++ project template in Meson - part 2"
date = "2024-01-28"
author = "SteelPh0enix"
authorTwitter = "steel_ph0enix"
tags = ["c", "cpp", "meson", "guide"]
keywords = ["c", "cpp", "c++", "meson", "project", "template", "getting", "started", "guide"]
description = "Last time we got the basics, now we get the tests"
showFullContent = false
+++


## Intro

In [previous part](/posts/making-c-cpp-project-template-in-meson-part-1), we've created the base of our project by defining it's structure and making our first modules and executable.
In this one, we're gonna add unit tests support.

## Harnessing the power of unit tests

I want to assume that You have at least minimal experience with testing your code, but i know some people who don't, so i will not.
But I'll also try not to over explain, there are other resources for that.
If you already know a thing or two about unit testing, you can ignore the next paragraph.
Or maybe don't, maybe you'll learn something new.
I dunno.

When You write some code, it's usually a good habit to test it.
After all, *how do you know if the code working correctly?*
You run it, and see if it does what it's supposed to.
The simplest way of doing that is writing some small programs that use the code in question in various ways and check if the outputs of this code are correct.
For example, let's assume you've written a function that calculates the amount of damage an attack does in an RPG game.
To test it, You can create some attack scenarios with various items, statistics, and whatever else can influence damage.
Then, calculate the results manually beforehand and write some code that will perform those scenarios using this function and check if the numbers are correct.
Tests that verify those small pieces of code, like functions or classes, are usually called **unit tests**.
Usually a **unit test harness** is used for testing - a library that provides utilities for creating, running, and gathering the results of those tests.
