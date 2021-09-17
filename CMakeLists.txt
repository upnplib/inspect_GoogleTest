# Copyright (C) 2021 GPL 3 and higher by Ingo Höft,  <Ingo@Hoeft-online.de>
# Redistribution only with this Copyright remark. Last modified: 2021-09-17

# Test build with:
# cmake -S . -B build -G "NMake Makefiles" --log-level=DEBUG -D CMAKE_BUILD_TYPE=MinSizeRel -D BUILD_SHARED_LIBS=ON

cmake_minimum_required(VERSION 3.18)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(INSPECT_GOOGLETEST VERSION 0001)


if(NOT CMAKE_BUILD_TYPE)
    # If not specified select first build type from the list of all available
    # types. It is Debug if using default initialization from cmake.
    list(GET CMAKE_CONFIGURATION_TYPES 0 cmakeBuildType)
    set(CMAKE_BUILD_TYPE ${cmakeBuildType} CACHE STRING "Choose the type of build, options are: ${CMAKE_CONFIGURATION_TYPES}." FORCE)
    message(STATUS "No build type specified, defaulting to ${CMAKE_BUILD_TYPE}.")
endif()

message(STATUS "Generator ............. ${CMAKE_GENERATOR}")
message(STATUS "Configuration Types ... ${CMAKE_CONFIGURATION_TYPES}")
message(STATUS "Build Type ............ ${CMAKE_BUILD_TYPE}")

message(CHECK_START "Download and configue Googletest")

include(FetchContent)

FetchContent_Declare(
    googletest
    GIT_REPOSITORY    https://github.com/google/googletest.git
    GIT_TAG           origin/master
    GIT_SHALLOW       ON
)
if(WIN32)
    # reference: build/_deps/googletest-src/googletest/README.md
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
endif()

FetchContent_MakeAvailable(googletest)

# Output with cmake option --log-level=DEBUG
message(DEBUG "  DEBUG: googletest_SOURCE_DIR is: ${googletest_SOURCE_DIR}")
message(DEBUG "  DEBUG: googletest_BINARY_DIR is: ${googletest_BINARY_DIR}")

message(CHECK_PASS "done")

# The fetched content is made available by building the libraries in a
# subproject using 'add_subdirectory()' (encapsulated in
# 'FetchContent_MakeAvailable()'). This is the reason why we cannot build the
# gtests direct here. These builds are executed before the build in the
# subproject. Build of the gtests will fail because the libraries are not
# available yet. We have to use also a subproject for the gtests so we can
# control the order of the builds.
#
add_subdirectory(${PROJECT_SOURCE_DIR}/gtests ${PROJECT_SOURCE_DIR}/gtests/build)
