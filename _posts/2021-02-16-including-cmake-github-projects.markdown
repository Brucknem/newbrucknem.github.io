---
layout: post
title:  "Include CMake projects from Github"
date:   2021-02-16 15:51:00 +0100
categories: Coding
tags: tutorials cmake github coding
pin: false
toc: true
---

When I develop a CMake based project I often want to include other CMake projects into mine. Gladly CMake offers a quick and easy way on how to automatically pull the other project from [Github](https://github.com/), build it using its own `CMakeLists.txt` and make the results available to my own project.

## CMakeList.txt.in

To make CMake aware of the other project just add a `CMakeList.txt.in` and paste the following snippet into it. Keep in mind to replace the `<PARTS-IN-BRACKETS>` with your own values. An [example](#example) can be found at the end.

```cmake
cmake_minimum_required(VERSION 2.8.12)

project(<OTHER-PROJECT> NONE)

include(ExternalProject)
ExternalProject_Add(<OTHER-PROJECT>
  GIT_REPOSITORY    <LINK-TO-GIT-REPOSITORY>
  GIT_TAG           <COMMIT-OR-BRANCH-TO-USE>
  SOURCE_DIR        "${CMAKE_CURRENT_BINARY_DIR}/<OTHER-PROJECT>-src"
  BINARY_DIR        "${CMAKE_CURRENT_BINARY_DIR}/<OTHER-PROJECT>-build"
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     ""
  INSTALL_COMMAND   ""
  TEST_COMMAND      ""
)
```
## CMakeList.txt

Now that CMake is aware of the project to include we can start building and including it. To do so add the following snippet to your projects `CMakeLists.txt`. Again, keep in mind to replace the `<OTHER-PROJECT>` with the value you chose in the `CMakeList.txt.in`. An [example](#example) can be found at the end.

```cmake
# Download and unpack the other project at configure time
configure_file(CMakeLists.txt.in <OTHER-PROJECT>-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/<OTHER-PROJECT>-download)
if (result)
    message(FATAL_ERROR "CMake step for <OTHER-PROJECT> failed: ${result}")
endif ()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/<OTHER-PROJECT>-download)
if (result)
    message(FATAL_ERROR "Build step for <OTHER-PROJECT> failed: ${result}")
endif ()

# Add the other project directly to our build.
add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/<OTHER-PROJECT>-src
        ${CMAKE_CURRENT_BINARY_DIR}/<OTHER-PROJECT>-build
        EXCLUDE_FROM_ALL)


### Here goes your normal project definition, source setup and linking

# Finally link the other project to our own project.
target_link_libraries(<YOUR-OWN-PROJECT>
        ...
        <OTHER-PROJECT>
        )

```

# Example

Recently I included the [TinyXML-2](https://github.com/leethomason/tinyxml2) by [Lee Thomason](http://www.grinninglizard.com/) into my [Guided Research]({% post_url 2021-02-12-guided-research %}) project. Fortunately he provides a `CMakeLists.txt` which enabled me to build and include it automatically using my CMake setup. 

## CMakeList.txt.in

```cmake
cmake_minimum_required(VERSION 3.10)

project(tinyxml2 NONE)

include(ExternalProject)
ExternalProject_Add(tinyxml2
  GIT_REPOSITORY    https://github.com/leethomason/tinyxml2.git
  GIT_TAG           master
  SOURCE_DIR        "${CMAKE_CURRENT_BINARY_DIR}/tinyxml2-src"
  BINARY_DIR        "${CMAKE_CURRENT_BINARY_DIR}/tinyxml2-build"
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     ""
  INSTALL_COMMAND   ""
  TEST_COMMAND      ""
)
```

## CMakeList.txt

```cmake
# Download and unpack tinyxml2 at configure time
configure_file(CMakeLists.txt.in tinyxml2-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/tinyxml2-download)
if (result)
    message(FATAL_ERROR "CMake step for tinyxml2 failed: ${result}")
endif ()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/tinyxml2-download)
if (result)
    message(FATAL_ERROR "Build step for tinyxml2 failed: ${result}")
endif ()

# Add tinyxml2 directly to our build.
add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/tinyxml2-src
        ${CMAKE_CURRENT_BINARY_DIR}/tinyxml2-build
        EXCLUDE_FROM_ALL)

### Here goes your normal project definition, source setup and linking

# Finally link the other project to our own project.
target_link_libraries(myProject
        ...
        tinyxml2
        )

```

# References

- [CMake](https://cmake.org/)
- The structure on how to include code from Github via CMake originates from the [GoogleTest](https://github.com/google/googletest/blob/master/googletest/README.md) framework.
- Example project:
  - [TinyXML-2](https://github.com/leethomason/tinyxml2)
  - [Lee Thomason](http://www.grinninglizard.com/)