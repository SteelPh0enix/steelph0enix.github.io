+++
title = "Making C/C++ project template in Meson - part 1"
date = "2024-01-27"
author = "SteelPh0enix"
authorTwitter = "steel_ph0enix"
tags = ["c", "cpp", "meson", "guide"]
keywords = ["c", "cpp", "c++", "meson", "project", "template", "getting", "started", "guide"]
description = "We're cooking a reasonable Meson template for C/C++ projects"
showFullContent = false
+++

## Intro

One of the things that I dislike about C and C++ in particular is lack of standardized build environment, package manager, and all that stuff.
And although CMake is de-facto "standard" over here, and even tries to double as a "package manager", I find it hard to like this tool.
It's not a *bad* tool, I've used it few times, I've seen some non-trivial projects made with CMake, but I did not like what I've seen.

I've also been missing a proper C/C++ project template for a very long time.
I was not aware I'm missing one, until i got a hang of the codebase of one of the projects I've been working on in my work, and experienced the true power of scripting and automation.
I've been tinkering with many build systems over the years, but I've never found one that really "clicked" for me.
And since I still cannot lift the curse of C/C++ from my life, it seems the only alternative I have is to pick *something* and force myself through.
And I'm taking You for a ride with me.

So we're going with [Meson Build system](https://mesonbuild.com/), which was recommended to me by a colleague few years ago.
Since then I've read the docs and also experimented with it a bit, but I haven't really made a proper project based on it yet, as of writing this sentence.
This is about to change.

I will try to make this series of blog posts as "followable" as possible, so You should be able to reproduce most of my work by yourself, and understand the choices I make.
Remember that the best tool for the job is sometimes the one you make yourself, not the one you blindly copy&paste without deeper understanding.
~~But please, don't reinvent the wheel if that's not necessary.~~
~~Use the right tools for the job, and whatnot.~~

## Goals

Our main goal here is to **make a generic, compiler/platform-independent, easy-to-expand project template in Meson for small-to-medium size C/C++ projects**.
I also want it to have some specific features that I believe are "must have" in any self-respecting project.
In the end, we will use that template to write some kind of application and maybe play with the project's structure afterwards, until I'm satisfied with the results.
After that, I have a plan to use this template as a basis for ARM Cortex-M project template, but we'll cross that bridge when we'll get to it.

The first thing that I'm interested in is unit and integration test support.
I don't want to make assumptions about test harnesses though, so we'll just have to make sure that the project's structure is *testable* and Meson can build the tests.
Unit tests have built-in support in Meson, so let's go with that.
There's also some support for code coverage, and we will surely explore that too.
Integration tests are much more project-dependent, so the only requirements is that Meson should recognize them as targets, but running them might be done via external script or program (executed by Meson).
Meson is our *build* system, not necessarily the runner, nor the harness, so let's not expect it to be able to handle generic integration tests by itself.
Some external tooling/scripting may still be required (especially when we'll get to the embedded part...), but everything should eventually be held together by Meson.

The second thing that I wanna see there is support for C/C++ LSP.
My primary choice there is `clangd`, so we'll have to generate `compile_commands.json` using Meson. Fortunately, it does that automatically.

The third thing is support for code checks and code formatting.
External tools can be plugged to Meson via [custom build targets](https://mesonbuild.com/Custom-build-targets.html), and we will configure at least one code checker and autoformatter like that.
Our project should provide commands to perform project-wide code and formatting check (which is very useful in git prehooks).

And the last, but not the least - documentation.
I want to be able to generate docs for my code via Meson.
Probably using Doxygen, as it's relatively easy to set up.
This also includes coverage reports, and we will most likely use `lcov`/`gcov` for that.

> *Did I mention that I've spent most of last year writing Rust code?*
> *Did you know that Cargo provides most of the stuff that I've described here out-of-the-box?*
> *Now you do.*

## Preparations

Before we start, let's check our prerequisites, because there's gonna be some.

- Meson - on Linux, should be in your package manager. If it's not, i assume you know what to do. On Windows, i recommend grabbing an installer from [Github](https://github.com/mesonbuild/meson/releases). Just make sure it's in your `PATH` variable - check if opening a terminal and running `meson --version` returns expected version string. I'm currently running 1.3.1. *Technically Meson is available via `winget`, but their repo currently provides outdated version, so i recommend installing it manually on Windows.*
- Ninja - I'll describe it's purpose in a bit. It should be bundled with Meson on Windows, and should be installed automatically on Linux when installing Meson via package manager. Should be in your `PATH` too. In any case, verify if it's installed correctly by running `ninja --version`. I'm currently running 1.11.1. You can also install it manually if needed [by downloading a zip from Github](https://github.com/ninja-build/ninja/releases) and extracting it into a directory that's in `PATH` variable.
- C/C++ toolchain - **TL;DR install latest GCC**. In theory, we don't have to choose any specific toolchain, because we're using Meson and Ninja which support the popular ones. **However**, I've mentioned that I want to have coverage report generation, and the only coverage tools that I'm familiar with are `lcov`/`gcov`, and this pretty much forces me to use GCC. If you don't care about code coverage reports, pick your treat. If you somehow got here as a complete newbie and you don't have any C/C++ toolchain installed, either grab GCC from your package manager if you're running Linux (look for package called `build-essentials` or `base-devel` or something like that), or - if you're running Windows - [download latest WinLibs package and add it's `bin` subdirectory to PATH](/posts/vscode-cpp-setup/#cc-toolchain). Or install Microsoft's Visual C++ toolchain via Visual Studio Build Tools (you may need to do that even after installing WinLibs, if you want to use `clang`, but I recommend **not** using MSVC for pure C unless you don't value your sanity).
- LLVM tools - mostly `clangd`, but we're also gonna use `clang-tidy` at some point. If you're running Linux, again - find those tools in your package manager. If you're running Windows and WinLibs, you already got them. If you don't use WinLibs, install latest LLVM release (`winget install llvm`, should be "recent enough", or get it from [Github](https://github.com/llvm/llvm-project/releases)), it should contain everything we'll need. Again, if manually installed - make sure it's in PATH by running `clangd --version`. I'm currently using 17.0.5.
- Doxygen - we're not gonna be using it any time soon, but let's make sure it's available. Linux - install from repo, Windows - it's already in WinLibs, if not using WinLibs (or you want the latest version), install manually - `winget install doxygen` or grab installer from [here](https://www.doxygen.nl/download.html). Run `doxygen --version` to verify, I'm using 1.10.0. **Note - Winget may not add it to PATH, it's installed in `Program Files/doxygen` by default. Add `bin` subdirectory to `PATH` manually if that's the case.**

Oh, and there's a small issue of `lcov` not having official Windows support. There are some unofficial releases, but we're not gonna use them - instead, we're gonna lock the coverage report generation feature to Linux and GCC only.
I don't want to completely lock this template to GCC because of that feature, so we will have to separate this part appropriately.

> We can always use containers to get this running with all features on Windows.

Also; i *assume* that we're gonna be using Ninja as our building "back-end", but it doesn't really matter because Meson should handle whatever "back-end" you'd like to use.
Just make sure to check the "whatever "back-end" you'd like to use" output when I'm talking about Ninja output.
This is an "I assume you know what you're doing" warning.

## Hello, world!

Assuming that everything is installed, we are ready to get going.
I've made a [repository](https://github.com/SteelPh0enix/meson_c_cpp_project_template) for this project, I'll try to make a tag after each part so You can easily follow.

Let's start by initializing a new Meson project in an empty directory.
The arguments don't really matter here - we're gonna be building our `meson.build` from scratch anyway, but right now we want to verify if our environment is set up correctly, so let's go with those for now:

```sh
meson init --name project_template --language cpp --type executable
```

This should generate two files - `meson.build` and `project_template.cpp`. `meson.build` should contain project definition with some reasonable defaults, executable, and a test:

```meson
project('project_template', 'cpp',
  version : '0.1',
  default_options : ['warning_level=3',
                     'cpp_std=c++14'])

exe = executable('project_template', 'project_template.cpp',
  install : true)

test('basic', exe)
```

Source file should contain a very simple test program:

```cpp
#include <iostream>

#define PROJECT_NAME "project_template"

int main(int argc, char **argv) {
    if(argc != 1) {
        std::cout << argv[0] <<  "takes no arguments.\n";
        return 1;
    }
    std::cout << "This is project " << PROJECT_NAME << ".\n";
    return 0;
}
```

Let's try building it.
But first, we need to generate actual build scripts via Meson, as it's (similarly to CMake) a meta-build system.
Which means that it's only a *generator* for an actual build system (like Ninja, or GNU Make) that performs the heavy lifting.
If you're not used to this approach, it may make no sense at first - why use two tools instead of one, right?
Unfortunately, building stuff is not easy in C/C++, especially when you're working on a big project with multiple dependencies, that must be supported on many different platforms.
As far as I know, the earliest successful application of this approach is [GNU Autotools](https://en.wikipedia.org/wiki/GNU_Autotools) project with it's `configure`-`make`-`make install` routine.
Apparently, it's good to have an additional layer of abstraction over the target build system.
I have already mentioned some reasons for that (multi-platform, yadda yadda yadda), also - meta-build systems are usually easier to use and have more features.
Anyway, to generate a build directory with all the stuff required for compilation, run this from project's directory

```sh
meson setup builddir
```

The argument `builddir` is the name of our build directory.
If you've set up your compiler and Ninja correctly, you should get some info about compiler executables and your environment, and `builddir` directory should appear with some Meson and Ninja files.

```
PS> meson setup builddir
The Meson build system
Version: 1.3.1
Source dir: F:\Projects\C_C++\meson_c_cpp_project_template
Build dir: F:\Projects\C_C++\meson_c_cpp_project_template\builddir
Build type: native build
Project name: project_template
Project version: 0.1
C++ compiler for the host machine: ccache c++ (gcc 13.2.0 "c++ (MinGW-W64 x86_64-ucrt-posix-seh, built by Brecht Sanders) 13.2.0")
C++ linker for the host machine: c++ ld.bfd 2.41
Host machine cpu family: x86_64
Host machine cpu: x86_64
Build targets in project: 1

Found ninja-1.11.1.git.kitware.jobserver-1 at C:\gcc\bin\ninja.EXE
```

Now, we can tell Meson to compile the project:

```sh
meson compile -C builddir
```

> We could also use `ninja` directly, but I'll stick to using Meson whenever possible instead, for compatibility reasons.
> That's actually a core part of this template - we **want** Meson to do stuff for us, we don't want to care what happens under the hood.

We should get some output from Ninja.

```
PS> meson compile -C builddir
ninja: Entering directory `F:/Projects/C_C++/meson_c_cpp_project_template/builddir'
[2/2] Linking target project_template.exe
```

And an executable

```
PS .> cd .\builddir\

PS .\builddir> .\project_template.exe
This is project project_template.

PS .\builddir> .\project_template.exe hello world
F:\Projects\C_C++\meson_c_cpp_project_template\builddir\project_template.exe takes no arguments.
```

And also a test!

```
meson test
```

Oh, if you haven't noticed yet - the `-C` argument changes the working directory of Meson, so it's necessary only when you're outside of it.
I assume that we `cd`'d into it in previous step, so we don't need it now.
I'll also stop adding it to future example invocations, just remember that it exists and that Meson's working directory must be the one that was generated via `meson setup` for most commands.

Let's look at the output:

```
PS .\builddir>meson test
ninja: Entering directory `F:\Projects\C_C++\meson_c_cpp_project_template\builddir'
ninja: no work to do.
1/1 basic        OK              0.01s

Ok:                 1
Expected Fail:      0
Fail:               0
Unexpected Pass:    0
Skipped:            0
Timeout:            0

Full log written to F:\Projects\C_C++\meson_c_cpp_project_template\builddir\meson-logs\testlog.txt
```

Yeah, that checks out, we have one "test" and it does nothing and returns 0, so it passes.
If you have encountered any issues until now, take a break and investigate.
Check if everything is where it's supposed to be.
You can also run `meson` commands with `--help` flag to see what options you have available, just as an exercise.
Continue, when you're ready.

## Structuring our project

Now it's time to think about the structure of our project.
We must be aware of the fact that there's no one "best" project structure that would suit every kind of project.
However, due to the fact that I'd rather not work on a completely abstract project, we'll have to make some assumptions about it's structure, at least for now.

Let's gather what we already assumed about our project.
We assumed that it's supposed to be testable.
Unit tests are the bare minimum that we want to support.
We also assumed that we'd like our code to be checked by external tools, but we can ignore that, because those tools should work no matter how we structure our project.
It's just the matter of invoking them properly.
Same thing with Doxygen.
As for `clangd`, Meson provides `compile_commands.json`, therefore it will work as long as the structure is valid.

Considering the usual good design practices, we'll probably want to split our codebase into modules according to their responsibility.
No, not [*those*](https://en.cppreference.com/w/cpp/language/modules) modules, let's ignore the existence of C++20 for now.
We can store each module in a separate directory, with it's own `meson.build` file that would define how it's built.
Modules can be built as static libraries and linked to executables.
That way, it's trivial to test them - we can just link the same binary that's used with our program executable to the test executable, and validate it's behavior.

> This is how it's done in some projects that I've been working on in my current job, and if it's good enough choice for space-grade projects, it's sure as hell good enough choice for me.
> But really, I've seen this approach in action and it should work well.

Let's also assume that our project can contain more than one executable, because it should be simple to setup for that.
We're gonna put them in `apps` directory.
Modules will go to `lib`, and tests into `tests`.

Let's create some dummy libs, tests for them, and an application to tie it together.
Also; let's add `meson.build` to each directory, empty - for now.
This is how it might look like:

```
meson_c_cpp_project_template
â”‚   meson.build
â”‚
â”œâ”€â”€â”€apps
â”‚   â””â”€â”€â”€hello_world
â”‚           hello_world.cpp
â”‚           meson.build
â”‚
â”œâ”€â”€â”€lib
â”‚   â”œâ”€â”€â”€calc
â”‚   â”‚       calc.cpp
â”‚   â”‚       calc.hpp
â”‚   â”‚       meson.build
â”‚   â”‚
â”‚   â””â”€â”€â”€greeter
â”‚           greeter.cpp
â”‚           greeter.hpp
â”‚           meson.build
â”‚
â””â”€â”€â”€tests
    â”œâ”€â”€â”€calc
    â”‚       calc_test.cpp
    â”‚       meson.build
    â”‚
    â””â”€â”€â”€greeter
            greeter_test.cpp
            meson.build
```

### Adding libraries

Let's ignore the fact that we don't have a test harness yet, and focus on making the libraries build.
But first, we have to put some code into them:

calc.hpp:

```cpp
#pragma once

double celsius_to_fahrenheit(double celsius);
double fahrenheit_to_celsius(double fahrenheit);
```

calc.cpp:

```cpp
#include "calc.hpp"

double celsius_to_fahrenheit(double celsius) {
    return (celsius * (9.0 / 5.0)) + 32.0;
}

double fahrenheit_to_celsius(double fahrenheit) {
    return (fahrenheit - 32.0) * (5.0 / 9.0);
}
```

greeter.hpp:

```cpp
#pragma once
#include <string>

std::string greet(std::string const& name);
```

greeter.cpp:

```cpp
#include "greeter.hpp"

std::string greet(std::string const& name) {
    return std::string("Hello ") + name;
}
```

And now, let's fill `meson.build` files.
In order to build a library, we have to use `library()` function. Duh.

```meson
calc = library('calc', 'calc.cpp')
```

The first argument is name of the library, followed by 0 or more sources in next arguments.
There are some options that we can set here using keyword arguments, but we're gonna ignore them for now because we don't need any.
We also save the result of this function to `calc` variable, which we're gonna use later.
We need to do the same thing for the other module, and the next step is modifying our top-level `meson.build` to recognize this dependency.
Let's remove the `executable()` and `test()` calls generated by Meson, and call `subdir` instead, to include `lib` directory.

```meson
project('project_template', 'cpp',
  version : '0.1',
  default_options : ['warning_level=3',
                     'cpp_std=c++14'])

subdir('lib')
```

Then, we're gonna create `meson.build` in `lib`, that will include all the modules we currently have.

```meson
subdir('calc')
subdir('greeter')
```

And now, we should be able to build them.
I recommend removing `builddir` directory after every big change, although I'm pretty sure that there is a way to do it in more elegant fashion (re-`setup`?), and re-generating the build files.
Oh, yeah, it's `meson setup --reconfigure builddir`.
There you go.
And I got instantly reminded why I'm not using it - it doesn't really reconfigure *everything*, so it's better to stick to the ol' reliable `rm builddir` technique, at least until our project's structure stabilizes a bit.

```
PS> meson setup builddir
The Meson build system
Version: 1.3.1
Source dir: F:\Projects\C_C++\meson_c_cpp_project_template
Build dir: F:\Projects\C_C++\meson_c_cpp_project_template\builddir
Build type: native build
Project name: project_template
Project version: 0.1
C++ compiler for the host machine: ccache c++ (gcc 13.2.0 "c++ (MinGW-W64 x86_64-ucrt-posix-seh, built by Brecht
Sanders) 13.2.0")
C++ linker for the host machine: c++ ld.bfd 2.41
Host machine cpu family: x86_64
Host machine cpu: x86_64
Build targets in project: 2

Found ninja-1.11.1.git.kitware.jobserver-1 at C:\gcc\bin\ninja.EXE


PS> meson compile -C builddir
INFO: autodetecting backend as ninja
INFO: calculating backend command to run: C:\gcc\bin\ninja.EXE -C F:/Projects/C_C++/meson_c_cpp_project_template/
builddir
ninja: Entering directory `F:/Projects/C_C++/meson_c_cpp_project_template/builddir'
[4/4] Linking target lib/greeter/libgreeter.dll
```

And it seems that it works.
It detected 2 build targets in project, exactly what we wanted to see.
Let's look at `builddir` to confirm it.

```
meson_c_cpp_project_template\builddir\lib
â”œâ”€â”€â”€calc
â”‚   â”‚   libcalc.dll
â”‚   â”‚   libcalc.dll.a
â”‚   â”‚
â”‚   â””â”€â”€â”€libcalc.dll.p
â”‚           calc.cpp.obj
â”‚
â””â”€â”€â”€greeter
    â”‚   libgreeter.dll
    â”‚   libgreeter.dll.a
    â”‚
    â””â”€â”€â”€libgreeter.dll.p
            greeter.cpp.obj
```

Okay, but we got DLLs (or `.so`'s, if you're running Linux).
We don't want dynamic libs, we want static libs.
Fortunately, there's an easy fix, straight from the docs - `library()` respects the `default_library` project option, so we just have to change it in our project's settings before building the libs.

```meson
project('project_template', 'cpp',
  version : '0.1',
  default_options : {
    'warning_level': '3',
    'cpp_std': 'c++14',
    'c_std': 'c11',
    'default_library': 'static',
  }
)
```

I took an opportunity to refactor that ugly array list into a proper dict, and set the default C standard to C11.
You can find the list of available default options [here](https://mesonbuild.com/Builtin-options.html#base-options), if you're interested.
Let's rebuild the project (again, `--reconfigure` may not detect this change, so `rm builddir`), and as we can see - we've got static libs now!

```
meson_c_cpp_project_template\builddir\lib
â”œâ”€â”€â”€calc
â”‚   â”‚   libcalc.a
â”‚   â”‚
â”‚   â””â”€â”€â”€libcalc.a.p
â”‚           calc.cpp.obj
â”‚
â””â”€â”€â”€greeter
    â”‚   libgreeter.a
    â”‚
    â””â”€â”€â”€libgreeter.a.p
            greeter.cpp.obj
```

### Adding an executable

Next, let's use those libs with our executable.
Example `hello_world.cpp` may look like this:

```cpp
#include <calc.hpp>
#include <greeter.hpp>
#include <iostream>

int main() {
    std::cout << greet("random developer") << std::endl;
    std::cout << "12.5*F == " << fahrenheit_to_celsius(12.5) << "*C" << std::endl;
    std::cout << "12.5*C == " << celsius_to_fahrenheit(12.5) << "*F" << std::endl;
}
```

Of course, the LSP will not be able to detect those headers properly yet, so ignore all the errors.
Let the toolchain speak the truth.
This is the content of `hello_world`'s directory `meson.build`

```meson
hello_world = executable(
  'hello_world',
  'hello_world.cpp',
  link_with: [calc, greeter]
)
```

Notice that we've used previously defined `calc` and `greeter` variables.
In order to do that, `lib` subdirectory must be evaluated before `apps`, and that's why `subdir('apps')` call must be put below `subdir('lib')` in our main `meson.build` file.
`subdir` basically evaluates the `meson.build` from the directory provided via the argument, and retains the environment after that.
Pretty useful, but we have to be careful not to misuse that feature, otherwise our Meson scripts may become very fragile and complicated to maintain.

```meson
subdir('lib')
subdir('apps')
```

We also have to create `meson.build` for `apps` directory.

```meson
subdir('hello_world')
```

> And yes, we probably could just `subdir('apps/hello_world')`, but I wanna keep everything as local as possible.
> You may add `meson.build` to every directory of the project now, because we're gonna need them.

`meson setup builddir` tells me that there are 3 targets now.
That's what we want.
But after running `meson compile -C builddir`, it seems that we have a problem.

```
../apps/hello_world/hello_world.cpp:1:10: fatal error: calc.hpp: No such file or directory
    1 | #include <calc.hpp>
      |          ^~~~~~~~~~
compilation terminated.
```

Yeah, that checks out.
We've linked the libs, but we never told `executable()` where are their include files.
To do that, we need to use `include_directories` argument of `executable()`, which expects a list of objects created using `include_directories()` function.
And `include_directories()` accepts string with relative paths, so we can just add them to the `meson.build` of our libraries and use them just like the objects returned from `library()`.

```meson
calc_lib = library('calc', 'calc.cpp')
calc_includes = include_directories('.')
```

I've also added the suffix to lib.
We need to do the same for `greeter`, and fix our `hello_world`.

```meson
hello_world = executable(
  'hello_world',
  'hello_world.cpp',
  link_with: [calc_lib, greeter_lib],
  include_directories: [calc_includes, greeter_includes],
)
```

And voila, we should have a green build now!
Let's see if the executable actually works.

```
PS> .\builddir\apps\hello_world\hello_world.exe
Hello random developer
12.5*F == -10.8333*C
12.5*C == 54.5*F
```

Yeah, the output checks out.
And that is where I'd like to finish this part, because it's already long enough, but there's just one tiny thing that annoys me...

## Feeding the LSP

I assume that you're using an editor/IDE with LSP support, and will be using `clangd`.
If that's not the case, i strongly recommend getting one.
I personally use [neovim](https://neovim.io/), but I cannot really recommend it to everyone, so [Visual Studio Code](https://code.visualstudio.com/) with [clangd plugin](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd) should be more than enough.
You may want to look at my older guide to see how to setup that combination.
**You can find it [here](/posts/vscode-cpp-setup/#bonus-clangd-setup).**
Meson generates `compile_commands.json` by default, automatically on every build (i think).
However, `clangd` will not be aware of that, because this file will be stored in `builddir` instead of our project's root directory, so we have two options:

1. Tell `clangd` that `compile_commands.json` is in `builddir/`
1. Copy/link `compile_commands.json` into root directory of our project

Fortunately, we can go with the first solution, because `clangd` supports configuration, and has an option for telling it where the compilation database is.
Let's create `.clangd` file in the root of our project, and put it there:

```yaml
CompileFlags:
  CompilationDatabase: builddir/
```

You can also add some [additional config](https://clangd.llvm.org/config) for the LSP.

aaaand... It looks like it's working for me, so that's it, we're done here.
I've made a tag and release on [this project's repository](https://github.com/SteelPh0enix/meson_c_cpp_project_template/tree/part-1) if you want to go the easy route and just `git clone` everything we've done here.
See you next time.
