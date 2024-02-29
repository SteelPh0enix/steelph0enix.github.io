+++
title = "Making C/C++ project template in Meson - part 2.5"
date = "2024-02-29"
author = "SteelPh0enix"
authorTwitter = "steel_ph0enix"
tags = ["c", "cpp", "meson", "guide"]
keywords = ["c", "cpp", "c++", "meson", "project", "template", "getting", "started", "guide"]
description = "I've fixed CppUTest wrap!"
showFullContent = false
+++


## Fixing CppUTest wrap and contributing to WrapDB

As per [my previous post](/posts/making-c-cpp-project-template-in-meson-part-2/), [I have fixed](https://github.com/mesonbuild/wrapdb/pull/1398) the CppUTest wrap and it's now available on WrapDB!
Since this post series is strictly related to Meson, I want to talk a bit more about creating wraps and contributing there.
If you don't care, and just want to continue to the guide part - jump to the [next section](#adding-unit-tests-support).

I have started the process of creating this wrap by making a very basic example project with a single binary that uses CppUTest.
Then, I plugged in CppUTest as a normal subproject, and began rewriting the `meson.build` files, along with the options.
Eventually I've managed to get to a point where it compiled successfully on my setup, linked to an app, and I was able to run it.
Thinking that's good enough, I've started worrying about wrapping it properly.

There are few different kinds of wraps that Meson supports.
If the project supports Meson, CMake or Cargo it can be wrapped directly, either as an archive or a specific revision of code on a supported repository.
Otherwise, it's possible to provide an "overlay" patch that adds/replaces the files of the library in order to provide Meson support - that's what file-wraps are.

I've considered it before, but to be clear - CppUTest's primary build system is CMake, but I don't have much pleasant memories related to CMake and I'd prefer to avoid interacting with it.

I already had the files necessary to create a file-wrap, so all I needed to do was modifying `cpputest.wrap`... Right?
Wrong!
This is where I've spent an hour or two, thinking that I can host this on my Github repo instead of contributing to WrapDB.
And this is my first major complaint about Meson!

First thing I wanted to try was putting the wrap patch files on my Github repository, and pointing the wrap to it.
I've figured that this way should be supported, because it enables hosting wraps on private repositories, for example in company environments.
So i did that, created a release to have an archive, pointed the wrap to it, and... It did not work, because Meson kept putting the patch files in different directory than source files.

I was thinking that `directory` option in `.wrap` file defines the directory both archives will be extracted to, but that's not the case!
This is the option that defines where the **source archive** will be extracted to, but not the **patch archive**.
And the output directory for patch was dependent on... The name of my repository. Duh.
Effectively, I think that I'd have to name my repository with the wrap `cpputest` to make it working that way, and I'd rather not do that...
So I've abandoned this idea and proceeded to read [how to contribute to WrapDB](https://mesonbuild.com/Adding-new-projects-to-wrapdb.html).
Alas, I believe that this way of providing files for file-wraps should be supported.

The contribution itself was rather pleasant experience - after moving the wrap files to the WrapDB fork, I've made a pull request and waited for CI to pass.
I was happy to see the CI checked the code by running a Python script with no external dependencies, so I could run the checks locally.
I was even more happy to see that this script actually downloaded and built every library WrapDB supports, and CI was running it on Windows, Linux and MacOS runners with Clang, GCC and MSVC.
That is a nice thing, but as we experienced, it's not quite enough to guarantee quality - even practically unusable wraps can build with no issues.
I was less happy when I've enabled the CI on my fork and it ate 1/4th of my monthly free GitHub runner limit after a single full run.
Oh well.
Good thing they do incremental builds on PRs.

Of course, my PR it did not pass the CI first time - it forced me to add some functionality to the wrap (platform autodetection), but after that it was good to go and got merged.
The whole process took only a few hours, the PR was reviewed, the CI did it's job - no complains there.

## Adding unit tests support

Now we can proceed with the actual subject of this post.

Remove everything from the `subprojects/` directory, open a terminal and run `meson wrap install cpputest`.
This is the preferred method of installing wraps, and I forgot to mention it earlier.
I don't quite understand why it's not able to create `subprojects/` directory by itself by default, but it's a non-issue.

I assume that you've followed the previous part, and already added `tests` directory to main `meson.build`, and created example test in `tests/calc/`.
If not, [go back and do that](/posts/making-c-cpp-project-template-in-meson-part-2/#managing-external-dependencies-with-meson).
If you already did that, then the project should compile successfully.
The only thing I'd like to change is `cpputest_dependency` to just `cpputest`, because it looks better.

tests/meson.build:

```meson
cpputest_project = subproject('cpputest', required: true)
cpputest = cpputest_project.get_variable('cpputest_dep')

subdir('calc')
subdir('greeter')
```

tests/calc/meson.build:

```meson
calc_test_exec = executable('calc_test', 'test.cpp', dependencies: cpputest)
test('calc test', calc_test_exec)
```

Now, we should also be able to run our test via `meson test` command.

```
> meson test -C builddir
ninja: Entering directory `C:\Users\phoen\Projects\meson_c_cpp_project_template\builddir'  
ninja: no work to do.
1/1 calc test        FAIL            0.01s   exit status 1
>>> MALLOC_PERTURB_=245 ASAN_OPTIONS=halt_on_error=1:abort_on_error=1:print_summary=1 UBSAN
_OPTIONS=halt_on_error=1:abort_on_error=1:print_summary=1:print_stacktrace=1 C:\Users\phoen
\Projects\meson_c_cpp_project_template\builddir\tests\calc\calc_test.exe


Ok:                 0
Expected Fail:      0
Fail:               1
Unexpected Pass:    0
Skipped:            0
Timeout:            0

Full log written to C:\Users\phoen\Projects\meson_c_cpp_project_template\builddir\meson-log
s\testlog.txt
```


