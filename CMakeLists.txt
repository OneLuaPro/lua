# MIT License
#
# Copyright (c) 2023 Kritzel Kratzel.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of
# this software and associated documentation files (the "Software"), to deal in 
# the Software without restriction, including without limitation the rights to 
# use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
# the Software, and to permit persons to whom the Software is furnished to do so,
# subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all 
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
#
# ------------------------------------------------------------------------------
# CMake for Lua
# ------------------------------------------------------------------------------
# Setup with Visual Studio 17 2022 generator for Win32
# ------------------------------------------------------------------------------
# Visual Studio is a mutlti-configuration generator
# https://stackoverflow.com/questions/24460486/
#
# Basic instructions for out-of-source build
# ------------------------------------------
# mkdir build
# cd build
# cmake .. -G "Visual Studio 17 2022" -A Win32
# cmake --build . --config Release
# cmake --install . --config Release
# 
# Available architectures (-A ...) are: Win32, x64, ARM, ARM64

# ------------------------------------------------------------------------------
# General definitions
cmake_minimum_required(VERSION 3.25 FATAL_ERROR)
set(LUA_VERSION_MAJOR 5)
set(LUA_VERSION_MINOR 4)
set(LUA_VERSION_PATCH 6)
set(LUA_VERSION ${LUA_VERSION_MAJOR}.${LUA_VERSION_MINOR}.${LUA_VERSION_PATCH})

# ------------------------------------------------------------------------------
# Installation prefix directory - defaults to C:\Apps\lua-X.Y.Z
# Needs to be defined before projectdefinition statement - for whatever reason
if(NOT CMAKE_INSTALL_PREFIX)
  if(WIN32)
    set(CMAKE_INSTALL_PREFIX "C:/Apps/lua-${LUA_VERSION}")
  else()
    message(FATAL_ERROR "Not yet implemented.")
  endif()
endif()

# ------------------------------------------------------------------------------
# Project defintion
project(lua
  LANGUAGES C
  VERSION ${LUA_VERSION_MAJOR}.${LUA_VERSION_MINOR}.${LUA_VERSION_PATCH}
)
# Lua is completely ANSI-C (https://www.lua.org/faq.html#1.1)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_C_STANDARD_REQUIRED ON)

# ------------------------------------------------------------------------------
# Other settings
# set(CMAKE_VERBOSE_MAKEFILE ON)
if(NOT CMAKE_BUILD_TYPE)
  # Visual Studio generators irgore this setting
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()
# create valiable with upper-case content out of CMAKE_BUILD_TYPE
string(TOUPPER ${CMAKE_BUILD_TYPE} CMAKE_BUILD_TYPE_UC)

# ------------------------------------------------------------------------------
# Setup GNU-alike installatin directories
include (GNUInstallDirs)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})
set(INSTALL_PREFIX
  ${CMAKE_INSTALL_PREFIX} CACHE PATH "Installation prefix")
set(INSTALL_DIR
  ${CMAKE_INSTALL_PREFIX} CACHE PATH "Install base dir for package config.")
set(INSTALL_LIBDIR
  ${CMAKE_INSTALL_LIBDIR} CACHE PATH "Installation directory for libraries")
set(INSTALL_BINDIR
  ${CMAKE_INSTALL_BINDIR} CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDEDIR
  ${CMAKE_INSTALL_INCLUDEDIR} CACHE PATH "Installation directory for header files")
set(INSTALL_DOCDIR
  ${CMAKE_INSTALL_DOCDIR} CACHE PATH "Installation directory for documentation")
set(INSTALL_MANDIR
  ${CMAKE_INSTALL_MANDIR} CACHE PATH "Installation directory for manpages")
set(DEF_INSTALL_CMAKEDIR share/cmake/lib${PROJECT_NAME})
set(INSTALL_CMAKEDIR
  ${DEF_INSTALL_CMAKEDIR} CACHE PATH "Installation directory for CMake files")

# ------------------------------------------------------------------------------
# Report to user
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C compiler flags: ${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_${CMAKE_BUILD_TYPE_UC}}")
message(STATUS "Installation prefix: ${INSTALL_PREFIX}")
foreach(p BIN INCLUDE LIB DOC MAN CMAKE)
  file(TO_NATIVE_PATH ${CMAKE_INSTALL_PREFIX}/${INSTALL_${p}DIR} _path)
  message(STATUS "Installing ${p} components to ${_path}")
  unset(_path)
endforeach()

# ------------------------------------------------------------------------------
# Dive into subdirs
add_subdirectory(src)
add_subdirectory(doc)
