# scs CMake Buildsystem <a href="https://isocpp.org"><img src="https://img.shields.io/badge/version-2.1.2-blue" alt="version-2.1.2" /></a> <a href="./LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="Size" /></a>

## Overview

A replacement buildsystem for [scs](https://github.com/cvxgrp/scs). Thanks to this [`CMake`](http://cmake.org) buildsystem, scs can be easily compiled and linked by other `CMake` projects.

## Dependencies

`lapack` and `blas` are required to solve  [LPs](https://en.wikipedia.org/wiki/Linear_programming), [SOCPs](https://en.wikipedia.org/wiki/Second-order_cone_programming), [SDPs](https://en.wikipedia.org/wiki/Semidefinite_programming) with scs.

On Ubuntu `lapack` and `blas` are distributed by `apt`

```
sudo apt-get install libblas-dev liblapack-dev
```

while on macOS, they can be easily installed with `brew`

```
brew install lapack openblas
```

## Usage

How to use this buildsystem

1. Clone the repository

   ```
   git clone git://github.com/dic-iit/scs-cmake-buildsystem
   ```

2. Build it

   ```
   cd scs-cmake-buildsystem
   mkdir build
   cd build
   cmake -DCMAKE_INSTALL_PREFIX:PATH=<custom-folder> ../
   make
   make install
   ```

### Some Interesting Cmake options

You may also want to compile the tests. In this case when you configure the project, please call the following command

   ```
   cmake -DCMAKE_INSTALL_PREFIX:PATH=<custom-folder> -DBUILD_TESTING:ON ../
   make
   ctest
   ```

By default the buildsystem will compile the library as `shared`. If you want to
compile it as `static`, please call the following command when you configure the project

```
cmake -DCMAKE_INSTALL_PREFIX:PATH=<custom-folder> -BUILD_SHARED_LIBS=OFF ../
make
```

By default, the buildsystem will clone `scs v2.1.2`. You can find the original repository in `scs-cmake-buildsystem/build/_deps/scs-src`.

## License

Materials in this repository are distributed under the following license:

> All software is licensed under the MIT License. See [LICENSE](./LICENSE) file for details.

## FAQ

### How the version is chosen?

The version of this `CMake` project is chosen in accordance of the original project. For the time being we support only [`scs v2.1.2`](https://github.com/cvxgrp/scs/releases/tag/v2.1.2).
