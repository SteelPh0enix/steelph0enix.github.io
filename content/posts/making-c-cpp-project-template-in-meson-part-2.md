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

But before that...

### Fixing our mistakes

So, uhh, let's look at our project.
Specifically, at our `hello_world.cpp`.
More specifically, at includes.
It's not that something is wrong with them *right now*, but think about it - what would happen if we'd put an `utils.hpp` file in both libraries?
That is actually a question that can be answered using the "fuck around and find out" method, so let's find out.

This is the content of our `utils.hpp`.
Adjust the returned string appropriately for the other module.

```cpp
#pragma once
#include <string>

inline std::string what_am_i() { return std::string("i am calc"); }
```

Now, our new `hello_world.cpp` will become this:

```cpp
#include <calc.hpp>
#include <greeter.hpp>
#include <utils.hpp>
#include <iostream>

int main() {
    std::cout << greet(what_am_i()) << std::endl;
    std::cout << "12.5*F == " << fahrenheit_to_celcius(12.5) << "*C" << std::endl;
    std::cout << "12.5*C == " << celcius_to_fahrenheit(12.5) << "*F" << std::endl;
}
```

Aaand... in my case, it prints `Hello i am calc`, which is caused by the fact that `calc_includes` are before `greeter_includes` in my `hello_world/meson.build`'s `executable()` call.
After swapping them, `Hello i am greeter` is printed.

Now that we know what happens, we can easely deduce that it's not supposed to work like that - because we don't have a reasonable way to access the other `utils.hpp`.
The simplest solution is to move the include directory one level up, to `lib` dir.
And in retrospect, that's how it should be done from the beginning, so excuse my blunder there.

In order to fix our mistake, we have to revisit `lib/meson.build`

also change `lib` to `libs`

## Harnessing the power of unit tests

I want to assume that You have at least minimal experience with testing your code, but i know some people who don't, so i will not.
But I'll also try not to over explain, there are other resources for that.
If you already know a thing or two about unit testing, you can ignore the rest of this chapter.
Or maybe don't, maybe you'll learn something new.
I dunno.

When You write some code, it's usually a good habit to test it.
After all, *how do you know if the code working correctly?*
You run it, and see if it does what it's supposed to.
The simplest way of doing that is writinghttps://itsfoss.community/t/stop-cat-abuse-sorry-if-it-is-old/8923 some small programs that use the code in question in various ways, and check if the outputs of this code are correct.
For example, let's assume you've written a function that calculates the amount of damage an attack does in a RPG game.
To test it, You can create some attack scenarios with various items, statistics, and whatever else can influence damage.
Then, calculate the results manually beforehand, and write the code that will perform those calculations using that function and verify if they match your calculations.
Tests that verify those small pieces of code, like functions or classes, are usually called **unit tests**.
The most basic unit test can be something like this:

```c
#include <assert.h>

#include <DamageCalculation.h>

#include "test_utils.h"

int main() {
    Weapon const weapon = create_test_weapon();
    Enemy const enemy = create_test_enemy();

    assert(calculate_damage(&weapon, &enemy) == 3.14);
}
```

Fortunately, the humanity have *(mostly)* progressed past the need for `assert`, and invented **test harnesses** that are *(usually)* less painful to use.
Trust me on that.
Even C guys didn't like rawdogging `assert`, and jumped on [Unity](http://www.throwtheswitch.org/unity) (not [*that*](https://www.axios.com/2023/09/22/unity-apologizes-runtime-fees) Unity) and other similar "wrappers".
In any case, using a test harness is easier than making one ourselves (or not using one at all), so we are going to use one.
I'd prefer if it also were multi-platform and supported both C and C++, and fortunately we have some options in the open-source market for that.

## Managing external dependencies with Meson

Before we choose a test harness, however, let's check how Meson handles external dependencies.
Fortunately, [documentation](https://mesonbuild.com/Dependencies.html) got us covered.
And it seems that Meson supports *a lot* of popular libraries in very specific ways, which is very, very nice.
However, the `dependency()` function seems to work only for locally available packages, which would usually mean that we have to manually download and set up the libraries for development each time we set up a new project environment - but there's a nice catch!
Meson also provides [**Wrap dependency system**](https://mesonbuild.com/Wrap-dependency-system-manual.html).
I recommend reading the linked documentation, but if you want a TL;DR: this is a package manager.
Sort of.
You put a valid `.wrap` file in `subprojects/` directory and voila, Meson can download and build the dependency, allowing us to use it in our project - automagically.
And I really do hope that it works like that, because with C and C++ - if something sounds too good to be true, it usually isn't (or there's a *humongous* catch attached).

