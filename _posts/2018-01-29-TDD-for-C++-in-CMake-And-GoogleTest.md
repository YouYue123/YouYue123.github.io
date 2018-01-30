---
layout: post
section-type: post
title: TDD for C++ in CMake and GoogleTest
category: tech
tags: [ 'Test driven development','C++','CMake','googletest','OpenGL', 'Ubuntu']
---

## Goal

Setup a cross-platform Test driven development environment for C++ based on CMake building system

## Environment

- Local OS: macOS High Sierra Version 10.13.1
- Compiler in local OS: Apple LLVM version 9.0.0 (clang-900.0.39.2)
- Remote OS: Ubuntu Server 16.04 LTS on AWS EC2
- Compiler in remote OS: g++ (Ubuntu 5.4.0-6ubuntu1~16.04.5) 5.4.0 20160609
- CMake: 3.5.2

## Initiative for C++ TDD 

### 1. Seperate Test code from production

While developing the main product for Airsquire, in order to test some part of the code, team need to write lots of magic number in main.cpp. And commend out and in again and again for different test cases. This mixed the test code with production code which is a very bad practice to keep robust quality.

### 2. Sturcture code for better maintenance

Actually it is all about testilibity thinking. Before adopting TDD in Airsquire's codebase, team is not always clear for different class's responsibility and how to implement a class method in a testable manner. After adopting it, by the help of test coverage team knows which statement should be maintained or which method should be encaplsuled for good testibility. This really helps maintenance and code is much easier to read not only for machien but also for humanity.

In addition, good structure of C++ code can also take benefit from C++ buidling system. Recompling and linking massive code repeatly is really painful for a C++ programmer. (P.S.: If you see a C++ programmer is playing his cellphone, don't blame on him. He is waiting for the code to be complied). By seperating into good structure, it will reduce time by compiling only the neccessary part. 

### 3. No more "Segment Fault 11" or "Killed"

By setting up a good testing framework like [googletest](https://github.com/google/googletest), this helps the team to document different senario by writting code to explain instead of making ambiguous. And it also helps us to identify which part is getting problem especially the stack trace is not useful. And most of the time C++ runtime gives really comfusing or useless debug information like "Segment Fault 11" or "Killed". Sometimes without a test case to decrease scope, it will need 1 dude day to identify the issue.


## Setup procedure and Sample project

After reading through this [article](https://github.com/google/googletest/blob/master/googletest/docs/Primer.md), I decided to use googletest in my team for the following 3 main reasons:

1. Good documentation
2. Active and strong maintenance team
3. Strong community

Here I do not want to repeat the content in googletest's doc, I wanna to give you a quickstart in how to setup a project using googletest and CMake.

Here is the project structure, the source code can be found in my [github repo](https://github.com/YouYue123/GoogleTest-With-CMake)

    ├── build                     # build folder for this project
    ├── src
    │   ├── sample_lib_1          # Sample Library 1 folder
    │   │   ├── sample_lib_1.cpp  # Implementation file for Sample Library 1
    │   │   ├── sample_lib_1.hpp  # Declarition file for Sample Library 1
    │   │   ├── CMakeList.txt     # CMake definition file for Sample Library 1
    │   ├── sample_lib_2          # Same as above structure
    │   │   ├── sample_lib_2.cpp  
    │   │   ├── sample_lib_2.hpp  
    │   │   ├── CMakeList.txt    
    │   ├── CMakeList.txt         # CMake defination for the whole src folder
    │   └── main.cpp              # Main entrance
    ├── tests                
    │   ├── testSampleLib1        # Test Sample Lib 1 folder
    │   │   ├── CMakeLists.txt    # CMake defination for Test Sample Lib 1 folder
    │   │   ├── testSampleLib1.cpp# Test case implementation file 
    │   │   ├── testSampleLib1.hpp# Test case declaration file
    │   │   ├── main.cpp          # Entrance of this test case
    │   │   ├── testSampleLib1.cpp    
    │   ├── testSampleLib2        # Similar to testSampleLib1
    │   │   ├── ...
    │   │   ├── ...   
    │   │   ├── ...
    │   │   ├── ...
    │   │   └── ...
    │   └── CMakeLists.txt        # CMake defination for the whole tests folder
    └── CMakeLists.txt


Here I will eleborate the CMakeLists.txt structure for each detail funtionality.

### Overall CMakeLists.txt 

And outside there is a overall CMakeLists.txt which contains the following content. You can add your additional CMAKE Standard here if you wanna to use more advanced features for recent C++. Don't forget  **enable_testing()**. This will enable your **add_test()** under tests folder. The add_subdirectory will add CMakeLists.txt under src and tests into your project.

```bash
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(GoogleTestSampleProject)

set(CMAKE_CXX_STANDARD 14)
if(CMAKE_COMPILER_IS_GNUCXX)
    add_definitions(-std=gnu++0x)
endif()

enable_testing()

add_subdirectory(src)
add_subdirectory(tests)
```

### CMakeLists.txt under src

This file aggregates all of your sub-components and integrate them into the main entrance

```bash
add_subdirectory(sample_lib_1)
add_subdirectory(sample_lib_2)
add_executable (main main.cpp)
target_link_libraries (main SAMPLE_LIB_1 SAMPLE_LIB_2)
```


### CMakeLists.txt under each component

In each component this file defines all of the code need to be compiled and dependency for this component. And it also defines the exported signature.

```bash
set(SAMPLE_LIB_1_SRCS
    sample_lib_1.hpp
    sample_lib_1.cpp
)
# Declare the library
add_library(SAMPLE_LIB_1 STATIC
    ${SAMPLE_LIB_1_SRCS}
)
# Link dependency
target_link_libraries(SAMPLE_LIB_1
    SAMPLE_LIB_2
)
# Specify here the include directories exported by this library
target_include_directories(SAMPLE_LIB_1 PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
)
```


### CMakeLists.txt under tests

This CMakeLists is very important as recommended by [GoogleTest Doc](https://github.com/google/googletest/tree/master/googletest). This ues CMake to download GoogleTest as part of the build's configure step. This is just a little more complex, but doesn't have the limitations of the other methods.

```bash
# We need thread support
find_package(Threads REQUIRED)

# Enable ExternalProject CMake module
include(ExternalProject)

# Download and install GoogleTest
ExternalProject_Add(
    gtest
    URL https://github.com/google/googletest/archive/master.zip
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/gtest
    # Disable install step
    INSTALL_COMMAND ""
)

# Get GTest source and binary directories from CMake project
ExternalProject_Get_Property(gtest source_dir binary_dir)

# Create a libgtest target to be used as a dependency by test programs
add_library(libgtest IMPORTED STATIC GLOBAL)
add_dependencies(libgtest gtest)

# Set libgtest properties
set_target_properties(libgtest PROPERTIES
    "IMPORTED_LOCATION" "${binary_dir}/googlemock/gtest/libgtest.a"
    "IMPORTED_LINK_INTERFACE_LIBRARIES" "${CMAKE_THREAD_LIBS_INIT}"
)

# Create a libgmock target to be used as a dependency by test programs
add_library(libgmock IMPORTED STATIC GLOBAL)
add_dependencies(libgmock gtest)

# Set libgmock properties
set_target_properties(libgmock PROPERTIES
    "IMPORTED_LOCATION" "${binary_dir}/googlemock/libgmock.a"
    "IMPORTED_LINK_INTERFACE_LIBRARIES" "${CMAKE_THREAD_LIBS_INIT}"
)

# I couldn't make it work with INTERFACE_INCLUDE_DIRECTORIES
include_directories("${source_dir}/googletest/include"
                    "${source_dir}/googlemock/include")
          
add_subdirectory(testSampleLib1)
add_subdirectory(testSampleLib2)

```

### CMakeLists.txt under each test case

This file defines the test needed to be compiled and also link google test and google mock for your test case. **Be attention**, the main.cpp under each test case I implement a ConfigurableEventListener which extends TestEventListener to custimize the output.

```bash
file(GLOB SRCS *.cpp)
ADD_EXECUTABLE(testSampleLib1 ${SRCS})
TARGET_LINK_LIBRARIES(
    testSampleLib1
    SAMPLE_LIB_1
    libgtest
    libgmock
)
add_test(NAME testSampleLib1
         COMMAND testSampleLib1)
```

## How to use

### Build

First you need to make a build directory and build your project there

```bash
mkdir build
cd build
cmake ..
make
```

### Run All Tests

Under build folder you can use a simple commend to run all of the test cases

```bash
cd build && make test
``` 

The result will be 

```bash
Running tests...
Test project /Users/youyue/Downloads/gtest-cmake-example-master/build
    Start 1: testSampleLib1
1/2 Test #1: testSampleLib1 ...................   Passed    0.01 sec
    Start 2: testSampleLib2
2/2 Test #2: testSampleLib2 ...................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   0.02 sec
```

### Run individual test

By going to individual test folders unde **build/tests/${testCaseName}**, you can run individual test directly. And here you can use googletest command to control your test case like --gtest_list_tests, --gtest_repeat, --gtest_death_test_style. In addition running individual test, you will see the custimized implementation output from ConfigurableEventListener.

The result will be like

```bash
[==========] Running 1 test from 1 test case.
[==========] 1 test from 1 test case ran. (0 ms total)
[  PASSED  ] 1 test.
```