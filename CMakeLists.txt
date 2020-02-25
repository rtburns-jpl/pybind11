# CMakeLists.txt -- Build system for the pybind11 modules
#
# Copyright (c) 2015 Wenzel Jakob <wenzel@inf.ethz.ch>
#
# All rights reserved. Use of this source code is governed by a BSD-style
# license that can be found in the LICENSE file.

cmake_minimum_required(VERSION 3.12)

project(pybind11 CXX)

set(PYBIND11_INCLUDE_DIR ${CMAKE_CURRENT_LIST_DIR}/include)

# Add a CMake parameter for choosing a desired Python version
if(NOT PYBIND11_PYTHON_VERSION)
    set(PYBIND11_PYTHON_VERSION
        ""
        CACHE STRING "Python version to use for compiling modules")
endif()
find_package(Python ${PYBIND11_PYTHON_VERSION} REQUIRED COMPONENTS Development)

# extract project version from source
file(STRINGS "${PYBIND11_INCLUDE_DIR}/pybind11/detail/common.h"
     pybind11_version_defines
     REGEX "#define PYBIND11_VERSION_(MAJOR|MINOR|PATCH) ")
foreach(ver ${pybind11_version_defines})
    if(ver MATCHES "#define PYBIND11_VERSION_(MAJOR|MINOR|PATCH) +([^ ]+)$")
        set(PYBIND11_VERSION_${CMAKE_MATCH_1}
            "${CMAKE_MATCH_2}"
            CACHE INTERNAL "")
    endif()
endforeach()
set(${PROJECT_NAME}_VERSION
    ${PYBIND11_VERSION_MAJOR}.${PYBIND11_VERSION_MINOR}.${PYBIND11_VERSION_PATCH}
)
message(STATUS "pybind11 v${${PROJECT_NAME}_VERSION}")

# Build an interface library target:
add_library(pybind11 INTERFACE)
add_library(pybind11::pybind11 ALIAS pybind11) # to match exported target
target_include_directories(pybind11 SYSTEM INTERFACE ${PYBIND11_INCLUDE_DIR})
target_link_libraries(pybind11 INTERFACE Python::Python)

add_library(module INTERFACE)
add_library(pybind11::module ALIAS module)
if(NOT MSVC)
    target_compile_options(module INTERFACE -fvisibility=hidden)
endif()
target_link_libraries(module INTERFACE pybind11::pybind11)
if(APPLE)
    target_link_libraries(module INTERFACE "-undefined dynamic_lookup")
endif()
