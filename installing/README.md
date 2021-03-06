# Installing

This isn't going to be a strict technical definition of installing, I just want to distill roughly what it is, what you get from it, and why it's cool.

Installing is really just copying certain files from a src/build location to somewhere else so other projects can use them. Quite _where_ this was drove me nuts for ages (coming from a predominantly Windows background it just felt weird). It doesn't help that lots of examples are predominantly `make` focussed which makes understanding how things work on Windows more difficult (I use Visual Studio most of the time).

Before understanding how to install your own libraries it's actually really helpful to use an existing library that's already done all the hard work for you. Let's choose Google's [benchmark](https://github.com/google/benchmark) library as an example.

The `benchmark` docs do an okay job of explaining how to download and build it, but unfortunately they're `make` orientated again. Let's list the commands you'd need to have this work on *nix/macOS or Windows.

```bash
mkdir benchmark && cd benchmark
git clone https://github.com/google/benchmark.git ./
# Benchmark requires Google Test as a dependency. Add the source tree as a subdirectory.
git clone https://github.com/google/googletest.git googletest
mkdir build && cd build
# if *nix/macOS
cmake -DCMAKE_BUILD_TYPE=RELEASE ..
# elseif Windows
cmake -G "Visual Studio 15 2017 Win64" -Dgtest_force_shared_crt=ON ..\benchmark\ # for Visual Studio build files (.sln)
# endif
# if *nix/macOS
cmake --build . --target install
# elseif Windows
cmake --build . --config Release --target install
# endif
```

The project generation commands have some platform specific differences (which is a little bit of a pain), but fortunately CMake has platform agnostic commands to do basically everything else you need without having to resort to calling `make` or `msbuild` explicitly.

Say hello to:

```bash
cmake --build .
```

It doesn't matter if you've generated `make` build files, or `Visual Studio` build files (or any other type of build files for that matter), typing `cmake --build .` (from the build/ folder) will build the project for you.

The part that follows `--build .`

```bash
--target install
```

Will take the built files and then _install_ (copy) them to the default system location. This is `/usr/local/lib/`, `usr/local/include/`, `usr/local/bin/` etc. on *nix/macOS and `C:\Program Files\<lib name>` on Windows (please see the [Windows](/examples/README.md#Windows) section in the miscellaneous part of the [examples](/examples/) README for more details on this). The `include`, `lib`, `bin` folders all exist as subdirectories inside the main folder.

These locations are where CMake will actually look for libraries/packages when running the `find_package` command from a `CMakeLists.txt` file.

The part I haven't yet mentioned

```bash
--config Release
```

Is to instruct which configuration to build when using Visual Studio solution files. In the `make` example, when we first generate the build files we must pick the build type (`-DCMAKE_BUILD_TYPE=DEBUG/RELEASE` etc. the default is `DEBUG`) but with Visual Studio multiple build types are generated and you can select them at build time using `--config`. I might be getting some of the exact details wrong here but it's likely not far from the truth.

If you followed the above steps you should now have Google benchmark installed on your system. The incredibly useful thing once you've done this is now in your application you can simply use the `find_package` command to use `benchmark` as a dependency.

```bash
# CMakeLists.txt
cmake_minimum_required(VERSION 3.14)

project(my_app LANGUAGES CXX)

find_package(benchmark REQUIRED)

add_executable(${CMAKE_PROJECT_NAME} main.cpp)

# make sure we build using c++11
target_compile_features(${CMAKE_PROJECT_NAME} PRIVATE cxx_std_11)

target_link_libraries(${CMAKE_PROJECT_NAME}
    benchmark::benchmark
    benchmark::benchmark_main
)
```

And in your `main.cpp` file you can now `#include "benchmark/benchmark.h"` and use the library immediately without having to manually setup include paths and link directories and all the normal painful stuff you have to do use another library.

```c++
// main.cpp
#include "benchmark/benchmark.h"

void measure_something(benchmark::State& state)
{
    for (auto _: state)
    {
        // something interesting to measure...
    }
}

BENCHMARK(measure_something);
```

This blew my mind when it finally clicked because up until this point I'd been basically doing everything manually (calling `target_include_directories`, `target_link_directories` etc..). Being able to use `find_package` massively reduces the complexity of your `CMakeLists.txt` files when adding new libraries. The wrinkle is you have to hope maintainers have done this hard work for you (otherwise you can write FindXXX.cmake files but that's outside the scope of this intro).

This brings us finally to installing our own libraries. To understand exactly what's required I direct you to the example CMake [projects](/examples). If you start with [header-only](/examples/header-only) and read through the `CMakeLists.txt` I've added comments to each of the `install` commands, detailing what they do and why they're needed. Hopefully you should be able to lift the necessary parts for your own `CMakeLists.txt` files should you wish to provide the ability to install your own libraries.