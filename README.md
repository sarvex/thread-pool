<h1 align=center>
thread-pool
</h1>

[![say thanks](https://img.shields.io/badge/Say%20Thanks-👍-1EAEDB.svg)](https://github.com/DeveloperPaul123/thread-pool/stargazers)
[![Discord](https://img.shields.io/discord/652515194572111872?logo=Discord)](https://discord.gg/CX2ybByRnt)
![License](https://img.shields.io/github/license/DeveloperPaul123/thread-pool?color=blue)
![Release](https://img.shields.io/github/v/release/DeveloperPaul123/thread-pool)
![Documentation](https://img.shields.io/website?label=docs&url=https%3A%2F%2Fdeveloperpaul123.github.io%2Fthread-pool%2F)

[![Ubuntu](https://github.com/DeveloperPaul123/thread-pool/actions/workflows/ubuntu.yml/badge.svg)](https://github.com/DeveloperPaul123/thread-pool/actions/workflows/ubuntu.yml)
[![Windows](https://github.com/DeveloperPaul123/thread-pool/actions/workflows/windows.yml/badge.svg)](https://github.com/DeveloperPaul123/thread-pool/actions/workflows/windows.yml)
[![Style](https://github.com/DeveloperPaul123/thread-pool/actions/workflows/style.yml/badge.svg)](https://github.com/DeveloperPaul123/thread-pool/actions/workflows/style.yml)
[![Install](https://github.com/DeveloperPaul123/thread-pool/actions/workflows/install.yml/badge.svg)](https://github.com/DeveloperPaul123/thread-pool/actions/workflows/install.yml)

A simple, functional thread pool implementation using pure C++20.

## Features

* Built entirely with C++20
* Enqueue tasks with or without tracking results
* [High performance](#benchmarks)

## Integration

`dp::thread-pool` is a header only library. All the files needed are in `include/thread_pool`.

### CMake

`ThreadPool` defines two CMake targets:

* `ThreadPool::ThreadPool`
* `dp::thread-pool`

You can then use `find_package()`:

```cmake
find_package(dp::thread-pool REQUIRED)
```

Alternatively, you can use something like [CPM](https://github.com/TheLartians/CPM) which is based on CMake's `Fetch_Content` module.

```cmake
CPMAddPackage(
    NAME thread-pool
    GITHUB_REPOSITORY DeveloperPaul123/thread-pool
    GIT_TAG #0cea9c12fb30cb677696c0dce6228594ce26171a change this to latest commit or release tag
)
```

## Usage

Simple example:

```cpp
// create a thread pool with a specified number of threads.
dp::thread_pool pool(4);

// add tasks, in this case without caring about results of individual tasks
pool.enqueue_detach([](int value) { /*...your task...*/ }, 34);
pool.enqueue_detach([](int value) { /*...your task...*/ }, 37);
pool.enqueue_detach([](int value) { /*...your task...*/ }, 38);
// and so on..
```

You can see other examples in the `/examples` folder.

## Benchmarks

Benchmarks were run using the [nanobench](https://github.com/martinus/nanobench) library. See the `./benchmark` folder for the benchmark code. The benchmarks are set up to compare matrix multiplication using the `dp::thread_pool` versus other thread pool libraries. These include:

* [ConorWilliams/Threadpool](https://github.com/ConorWilliams/Threadpool)
* [bshoshany/thread-pool](https://github.com/bshoshany/thread-pool) (C++17)

The benchmarks are set up so that each library is tested against `dp::thread_pool` using `std::function` as the baseline. Relative measurements (in %) are recorded to compare the performance of each library to the baseline.

### Machine Specs

* AMD Ryzen 7 5800X (16 X 3800 MHz CPUs)
* 32 GB RAM

### Results


#### Summary

In general, `dp::thread_pool` is faster than other thread pool libraries in most cases. This is especially the case when `std::move_only_function` is available. `fu2::unique_function` is a close second, and `std::function` is the sloweset when used in `dp::thread_pool`. In certain situations, `riften::ThreadPool` pulls ahead in performance. This is likely due to the fact that this library uses a lock-free queue. There is also a custom semaphore and it seems that there is a difference in how work stealing is handled as well.

#### Details

Below is a portion of the benchmark data from the MSVC results:

| relative |               ms/op |                op/s |    err% |     total | matrix multiplication 256x256
|---------:|--------------------:|--------------------:|--------:|----------:|:------------------------------
|   100.0% |               94.98 |               10.53 |    1.3% |     17.15 | `dp::thread_pool - std::function`
|   102.8% |               92.43 |               10.82 |    0.8% |     16.44 | `dp::thread_pool - std::move_only_function`
|    99.0% |               95.98 |               10.42 |    0.8% |     17.27 | `dp::thread_pool - fu2::unique_function`
|    89.8% |              105.77 |                9.45 |    0.3% |     18.94 | `BS::thread_pool`
|    96.8% |               98.07 |               10.20 |    0.5% |     17.59 | `riften::Thiefpool`

If you wish to look at the full results, use the links below.

[MSVC Results](./benchmark/results/benchmark_results_msvc.md)

[Clang Results](./benchmark/results/benchmark_results_clang.md)

Some notes on the benchmark methodology:

* Matrix sizes are all square (MxM).
* Each multiplication is `(MxM) * (MxM)` where `*` refers to a matrix multiplication operation.
* Benchmarks were run on Windows, so system stability is something to consider (dynamic CPU frequency scaling, etc.).
* Relative

## Building

This project has been built with:

* Visual Studio 2022
* Clang `10.+` (via WSL on Windows)
* GCC `11.+` (vis WSL on Windows)
* CMake `3.21+`

To build, run:

```bash
cmake -S . -B build
cmake --build build
```

### Build Options

| Option              | Description                                                         | Default |
|:--------------------|:--------------------------------------------------------------------|:-------:|
| `TP_BUILD_TESTS`    | Turn on to build unit tests. Required for formatting build targets. |   ON    |
| `TP_BUILD_EXAMPLES` | Turn on to build examples                                           |   ON    |

### Run clang-format

Use the following commands from the project's root directory to check and fix C++ and CMake source style.
This requires _clang-format_, _cmake-format_ and _pyyaml_ to be installed on the current system. To use this feature you must turn on `TP_BUILD_TESTS`.

```bash
# view changes
cmake --build build/test --target format

# apply changes
cmake --build build/test --target fix-format
```

See [Format.cmake](https://github.com/TheLartians/Format.cmake) for details.

### Build the documentation

The documentation is automatically built and [published](https://developerpaul123.github.io/thread-pool) whenever a [GitHub Release](https://help.github.com/en/github/administering-a-repository/managing-releases-in-a-repository) is created.
To manually build documentation, call the following command.

```bash
cmake -S documentation -B build/doc
cmake --build build/doc --target GenerateDocs
# view the docs
open build/doc/doxygen/html/index.html
```

To build the documentation locally, you will need Doxygen and Graphviz on your system.

## Contributing

Contributions are very welcome. Please see [contribution guidelines for more info](CONTRIBUTING.md).

## License

The project is licensed under the MIT license. See [LICENSE](LICENSE) for more details.

## Author

| [<img src="https://avatars0.githubusercontent.com/u/6591180?s=460&v=4" width="100"><br><sub>@DeveloperPaul123</sub>](https://github.com/DeveloperPaul123) |
|:----:|
