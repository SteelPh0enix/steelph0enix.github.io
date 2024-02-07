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

Aaand... In my case, it prints `Hello i am calc`, which is caused by the fact that `calc_includes` are before `greeter_includes` in my `hello_world/meson.build`'s `executable()` call.
After swapping them, `Hello i am greeter` is printed.

Now that we know what happens, we can easily deduce that it's not supposed to work like that - because we don't have a reasonable way to access the other `utils.hpp`.
The simplest solution is to move the include directory one level up, to `lib` directory.
And in retrospect, that's how it should be done from the beginning, so excuse my blunder there.

In order to fix our mistake, we have to revisit some `meson.build` files.
First, let's create an include directory object in `lib/meson.build`.

```meson
libs_includes = include_directories('.')

subdir('calc')
subdir('greeter')
```

I've added it before the `subdir()` calls just in case some modules would depend on each other.
This is not something we want to do often, because for every dependency we will probably have to create a mock in order to properly test the modules that use them, but it's sometimes necessary.

After that, remove the `include_directories()` calls from `greeter` and `calc` modules `meson.build`, and fix the `executable()` call arguments in `hello_world/meson.build`.

```meson
hello_world = executable(
  'hello_world',
  'hello_world.cpp',
  link_with: [calc_lib, greeter_lib],
  include_directories: [libs_includes],
)
```

And fix the includes in `hello_world.cpp`.
We can also remove the `utils.hpp` and restore the original `hello_world.cpp`.

```cpp
#include <calc/calc.hpp>
#include <greeter/greeter.hpp>
#include <iostream>

int main() {
    std::cout << greet("random developer") << std::endl;
    std::cout << "12.5*F == " << fahrenheit_to_celcius(12.5) << "*C" << std::endl;
    std::cout << "12.5*C == " << celcius_to_fahrenheit(12.5) << "*F" << std::endl;
}
```

Also; let's rename our `lib/` directory to `libs/`, this is a very small change but my brain automatically tries to write `libs/` instead of `lib/` because there are *multiple* libraries there, so I think it reflects this directory's content better.
Remember to also change the argument of `subdir()` in the root `meson.build`.

Verify if the project still builds and the executable still works. Remove the `builddir` first, as we made pretty big change and I honestly don't expect Meson to correctly re-generate it.


## Harnessing the power of unit tests

I want to assume that You have at least minimal experience with testing your code, but i know some people who don't, so i will not.
But I'll also try not to over explain, there are other resources for that.
If you already know a thing or two about unit testing, you can ignore the rest of this chapter.
Or maybe don't, maybe you'll learn something new.
I dunno.

When You write some code, it's usually a good habit to test it.
After all, *how do you know if the code working correctly?*
You run it, and see if it does what it's supposed to.
The simplest way of doing that is writing some small programs that use the code in question in various ways, and check if the outputs of this code are correct.
For example, let's assume you've written a function that calculates the amount of damage an attack does in an RPG game.
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

Although I'd replace that magic number with some very approximate manual calculation, if possible.
Tests should not be cryptic - this is, but only because it's an unspecified example.

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

After looking at available [wraps](https://mesonbuild.com/Wrapdb-projects.html), and considering the fact that I'd like to use that template for ARM projects, I have decided I'm going with [CppUTest](https://github.com/cpputest/cpputest).
If you want to try another test harness, feel free to do that and experiment - but keep in mind that the setup process may vary a bit, depending on the library needs.
CppUTest is relatively simple, which is nice if you're looking for something without much complexity, but it won't provide as many features as some other harnessed - like [Catch2](https://github.com/catchorg/Catch2) or [Google Test](https://github.com/google/googletest).
I like simple, and it should probably make porting everything to ARM easier, so I'm going with CppUTest.

Let's create `subprojects/` directory and run `meson wrap install cpputest` (or your preferred test harness).
Installation should proceed automatically, and `subprojects/cpputest.wrap` file should appear.
Then, to use it, we have to declare a `subproject()` in one of our `meson.build` files.
Let's do that in our `tests/meson.build`.
We'll also add our tests subdirectories, since we're already editing that file.
Add them below the `subproject()` to make sure the test harness is available there.

```meson
cpputest_project = subproject('cpputest', required: true)
cpputest_dependency = cpputest_project.get_variable('cpputest_dep')

subdir('calc')
subdir('greeter')
```

Now, we also have to declare `tests` directory as a `subdir()` in main `meson.build`.
I've put it after `apps`, because integration tests may require built apps as dependencies to run.

```meson
subdir('libs')
subdir('apps')
subdir('tests')
```

And finally, we can check if this works by deleting `builddir`, and running `meson setup builddir` and `meson compile -C builddir`.
You should see some new logs after running `setup`:

```
Executing subproject cpputest 

cpputest| Project name: cpputest
cpputest| Project version: 4.0
cpputest| C++ compiler for the host machine: ccache c++ (gcc 13.2.0 "c++ (MinGW-W64 x86_64-ucrt-posix-seh, built by Brecht Sanders) 13.2.0")
cpputest| C++ linker for the host machine: c++ ld.bfd 2.41
cpputest| Build targets in project: 5
cpputest| Subproject cpputest finished.

Build targets in project: 5

project_template 0.1

  Subprojects
    cpputest: YES
```

That tells us Meson found the wrap and should've downloaded it.
Compilation should also take significantly longer, because our test harness must be compiled for the first time.
If that's the case, our first step is done.
Now, we have to actually write a test.
Create `test.cpp` file in `tests/calc/` and put this there:

```cpp
#include <CppUTest/CommandLineTestRunner.h>
#include <CppUTest/TestHarness.h>

TEST_GROUP(FirstTestGroup){};

TEST(FirstTestGroup, FirstTest) {
    FAIL("Fail me!");
}

int main(int ac, char** av) {
    return CommandLineTestRunner::RunAllTests(ac, av);
}
```

Now, let's make it compile. Open `tests/calc/meson.build` and put it there:

```meson
calc_test_exec = executable('calc_test', 'test.cpp', dependencies: cpputest_dependency)
test('calc test', calc_test_exec)
```

In theory, this should build just fine.
However, in practice...

```
[44/44] Linking target tests/calc/calc_test.exe
FAILED: tests/calc/calc_test.exe
"c++"  -o tests/calc/calc_test.exe tests/calc/calc_test.exe.p/test.cpp.obj "-Wl,--allow-shlib-undefined" "-Wl,--start-group" "subprojects/cpputest-4.0/src/CppUTestExt/libCppUTestExt.a" "-Wl,--subsystem,console" "-lkernel32" "-luser32" "-lgdi32" "-lwinspool" "-lshell32" "-lole32" "-loleaut32" "-luuid" "-lcomdlg32" "-ladvapi32" "-Wl,--end-group"
C:/gcc/bin/../lib/gcc/x86_64-w64-mingw32/13.2.0/../../../../x86_64-w64-mingw32/bin/ld.exe: tests/calc/calc_test.exe.p/test.cpp.obj: in function `TEST_FirstTestGroup_FirstTest_Test::testBody()':
F:\Projects\C_C++\meson_c_cpp_project_template\builddir/../tests/calc/test.cpp:7:(.text+0x11): undefined reference to `UtestShell::getCurrent()'
```

In practice, I get a massive linking error.
Gee, I wonder why.

After few minutes of investigation, I've reached the conclusion: the *"official"* CppUTest wrap is **very bad**.
And by that, I mean it's completely broken.
If you've used a different test harness, and it works (i know for a fact that Catch2 should, last i tried at least...) - congratulations, you can skip the next part!
For the rest of you, don't worry - we're fix that issue. It'll just take a bit longer for you, and much longer for me.

## Side quest - fixing CppUTest!

Let's look at it's build files, as everything is stored in `subprojects/cpputest-4.0`.
First thing that I've noticed was spelling error - the author used `extinctions` instead of `extensions` (in multiple places, so this was... Intentional?)
Then, I've noticed something worse.

```meson
cpputest_dep = declare_dependency(
    link_with : cpputest_lib,
    version : meson.project_version(),
    include_directories : cpputest_dirs)

if get_option('extinctions')
    cpputest_dep = declare_dependency(
        link_with : cpputest_ext_lib,
        version : meson.project_version(),
        include_directories : cpputest_dirs)
endif
```

You see this, right?
Obviously, a typo, someone forgot to add `_ext`, but one that completely breaks the wrap.
Extensions are enabled by default, by the way.
See `meson_options.txt` in CppUTest directory:

```meson
option('extinctions',
    type : 'boolean',
    value : true,
    description : 'Use the CppUTest extension library'
)
```

So, what happens when we fix that? We can do it pretty easily, just declare a second dependency, we don't even have to link the extensions because at this point we're not using them.
Rename `cpputest_dep` for extensions to `cpputest_ext_dep` and let's add the second dependency to `cpputest.wrap`:

```meson
if get_option('extinctions')
    cpputest_ext_dep = declare_dependency(
        link_with : cpputest_ext_lib,
        version : meson.project_version(),
        include_directories : cpputest_dirs)
endif
```

```
[provide]
cpputest = cpputest_dep
cpputest_ext = cpputest_ext_dep
```

And after clean compilation, unfortunately, this doesn't solve my issue.
I get multiple undefined references to platform-related functions, because apparently someone forgot to add platform-specific implementations that CppUTest provides as dependencies...

Well, the fix is *easy* - I just have to make a proper wrap myself.
To be continued, after I make it working.

>Yeah, I could jump the ship and just use different library, but *where's the fun in that?*
>I've already vibe checked that one.
>Also; there is no tag in the repository for now, I'm gonna make one after making that wrap and finishing up this part in next post(s).
