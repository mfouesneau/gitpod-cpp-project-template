# CPP project template for Gitpod and VSCode

A template project for C++ development using [CMake](https://cmake.org/) and [Conan](http://conan.io/) package manager.

## Gitpod

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/mfouesneau/blackbodystars)


Gitpod is a friendly online IDE very similar to VSCode ([Gitpod.io](https://gitpod.io/)
In particular, it provides an entire container-based platform (not saying Docker) and provides VSCode features such as pair-coding while still compiling and running codes. It's also very convenient to write/edit and run some codes rapidly.

This repository holds a very basic workspace to start a C++ project and to work with Gitpod.

ℹ️ **The rest of the README contains some notes on the template and project configuration**

### Table of contents

<!--ts-->
  - [Example Code](#example-code)
  - [Conan Package Manager](#conan-package-manager)
    - [Installing and Configuring Conan](#installing-and-configuring-conan)
    - [Using Conan with CMake](#using-conan-with-cmake)
  - [Workspace configuration](#workspace-configuration)
    - [Container image `.gitpod.Dockerfile`](#container-image-gitpod-dockerfile)
    - [Gitpod configuration `.gitpod.yml`](#gitpod-configuration-gitpod-yml)
  - [Launchers and Tasks Definitions (`.vscode/tasks.json`, `.vscode/launch.json`)](#launchers-and-tasks-definitions-vscode-tasks-json-vscode-launch-json)
<!--te-->

## Example Code

The example codes are basic
```c++
#include <iostream>
namespace example {
	void example1(){
	    std::cout << "Example 1 called. \n";
	}
};
```
But also show how to install and use external libraries with `conan`. I give the example of `xtensor` (https://github.com/xtensor-stack/xtensor)

```c++
#include <iostream>
#include "xtensor/xarray.hpp"
#include "xtensor/xio.hpp"
#include "xtensor/xview.hpp"
#include "example.hpp"

int main() {
    xt::xarray<double> arr1
    {{1.0, 2.0, 3.0},
    {2.0, 5.0, 7.0},
    {2.0, 5.0, 7.0}};

    xt::xarray<double> arr2
    {5.0, 6.0, 7.0};

    xt::xarray<double> res = xt::view(arr1, 1) + arr2;

    std::cout << res;
}
```

## Conan Package Manager


[Conan](http://conan.io/) is an Open Source package manager for C and C++ development. It allows developers to quickly and efficiently manage their packages and dependencies across platforms and build systems.

In this project, we use Conan to manage our dependencies and build the project with [CMake](https://cmake.org/).

### Installing and Configuring Conan

I use python installation

```shell
> pip install conan
```

You may have to look for the library you need in the [package center](https://conan.io/center/), e.g:
```shell
> conan search xtensor -r conan-center
Existing package recipes:

xtensor/0.21.2
xtensor/0.21.3
xtensor/0.21.4
xtensor/0.21.5
```

add your requirements to the `conanfile.txt` in the `requires` section, e.g,
```yaml
[requires]
 xtensor/0.21.5
 ```

 ### Using Conan with CMake

 add `cmake` in the `generator` section to the `conanfile.txt`, e.g,
```yaml
[generators]
 cmake
 ```

Update your `CMakeLists.txt` to include the conan libraries, for instance before the targets
```cmake
include(conanbuildinfo.cmake)
conan_basic_setup()
```
And for the relevant targets
```cmake
target_link_libraries(example ${CONAN_LIBS})
```

Finally, you will need to run `conan install .` before `cmake`
```shell
> conan install . && cmake .
```

## Workspace configuration

Gitpod configuration is very similar to a VSCode workspace. The main difference is that Gitpod is a container-based platform, which means you can define your container images.

### Container image `.gitpod.Dockerfile`

More information: https://www.gitpod.io/docs/config-docker/

By default, Gitpod uses a standard image called `gitpod/workspace-full`.
This default image comes pre-installed with Docker, Go, Java, Node.js, C/C++, Python, Ruby, Rust, PHP, and tools such as Homebrew, Tailscale, Nginx, and several more.

If this image does not include the tools you need for your project, you can provide a public Docker image or your Dockerfile.
Using this technique provides us with the flexibility to install the tools & libraries required for your project.

> This template starts with this image as it will make sure that the platform works (sometimes new Debian images introduce breaking changes; see documentation), and installs the HDF5 library.


### Gitpod configuration `.gitpod.yml`

Here is the minimal configuration I use, which tells Gitpod to
* use the provided image information,
* install `conan` and run the first build on start,
* and install some C/C++ convenient extensions for VScode.

```yaml
image:
  file: .gitpod.Dockerfile

tasks:
  - init: "pip install conan"
  - command: "conan install . && cmake ."

vscode:
  extensions:
    - webfreak.debug
    - ms-vscode.cmake-tools
```

## Launchers and Tasks Definitions (`.vscode/tasks.json`, `.vscode/launch.json`)

VScode based IDEs can do the everyday IDE tasks but need some primary learning curve to set things up.

I set up this template to provide a workflow that compiles the code and runs it (with debugging options).
* A task that compiles the main code with `CMake` and `Make`
```json
{
    "label": "DEBUG: cmake && make",
    "type": "shell",
    "command": "cd ${workspaceRoot} && cmake . -DCMAKE_BUILD_TYPE=Debug && make",
    "problemMatcher": [],
    "group": {
        "kind": "build",
        "isDefault": true
    }
},
```
ℹ️  **Note**: `${workspaceRoot}` is the root of the workspace.

:warning: **Note**: `-DCMAKE_BUILD_TYPE=Debug` is essential to make the debugger catch breakpoints etc. The cost is that the build is slower and performance lower. (I also added a task to build the _release_ version)

* A launch configuration that runs the executable with `GDB` after running the task mentioned above.
```json
{ "type": "gdb",
  "request": "launch",
  "name": "GDB: blackbodystars",
  "target": "${workspaceRoot}/bin/blackbodystars",
  "args": [],
  "cwd": "${workspaceRoot}",
  "valuesFormatting": "parseText",
  "preLaunchTask": "DEBUG: cmake && make"
},
```
The `CMakeLists.txt` tells where the compilation puts the `target` executable.

:warning: **Note**: In VScode, the default launcher is with `cppdbg`. The associated extension (`ms-vscode.cpptools`) is not part of the open extension library (open VSX). (will it be?). Still, I also provide the corresponding launcher in the configuration file to use offline or remotely with VSCode.