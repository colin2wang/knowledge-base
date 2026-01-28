# CMake

CMake is an open-source, cross-platform family of tools designed to build, test and package software. CMake is used to control the software compilation process using simple platform and compiler independent configuration files, and generate native makefiles and workspaces that can be used in the compiler environment of your choice.

## Basic Usage

### Generate Build Files with MinGW
```shell
# Use MinGW to build
cmake -G "MinGW Makefiles" -DCMAKE_SH="CMAKE_SH-NOTFOUND" ..
```

### Common CMake Commands
```shell
# Generate build files in current directory
cmake .

# Generate build files in a separate build directory
mkdir build && cd build
cmake ..

# Build the project
cmake --build .

# Build with specific configuration
cmake --build . --config Release

# Install the project
cmake --install .
```

## CMakeLists.txt Basics

```cmake
# Set minimum CMake version
cmake_minimum_required(VERSION 3.10)

# Project name
project(MyProject)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Add executable target
add_executable(MyApp main.cpp)

# Add include directories
target_include_directories(MyApp PUBLIC include)

# Add libraries
target_link_libraries(MyApp some_library)
```

## Useful Resources
- [CMake Official Documentation](https://cmake.org/documentation/)
- [CMake Tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)