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
# Compile Lua Sources
# ------------------------------------------------------------------------------

# ------------------------------------------------------------------------------
# Create static Lua library
add_library(lib${PROJECT_NAME} STATIC)
target_sources(lib${PROJECT_NAME}
  PRIVATE lapi.c lctype.c lfunc.c lmathlib.c loslib.c ltable.c lundump.c
  lauxlib.c ldblib.c lgc.c lmem.c lparser.c ltablib.c lutf8lib.c lbaselib.c
  ldebug.c linit.c loadlib.c lstate.c ltm.c lvm.c lcode.c ldo.c liolib.c
  lobject.c lstring.c lzio.c lcorolib.c ldump.c llex.c lopcodes.c lstrlib.c
  lapi.h ldebug.h ljumptab.h lobject.h lprefix.h ltm.h lundump.h
  ldo.h llex.h lopcodes.h lstate.h  lvm.h lcode.h lfunc.h llimits.h lopnames.h
  lstring.h lzio.h lctype.h lgc.h lmem.h lparser.h ltable.h 
  PUBLIC FILE_SET HEADERS
  FILES lua.h luaconf.h lualib.h lauxlib.h lua.hpp
)
target_include_directories(lib${PROJECT_NAME}
  PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
  "$<INSTALL_INTERFACE:${INSTALL_INCLUDEDIR}>"
)

# ------------------------------------------------------------------------------
# Create and link lua binaries
add_executable(lua lua.c)
add_executable(luac luac.c)
target_link_libraries(lua lib${PROJECT_NAME})
target_link_libraries(luac lib${PROJECT_NAME})

# ------------------------------------------------------------------------------
# Installation instructions
install(
  TARGETS lib${PROJECT_NAME} EXPORT lib${PROJECT_NAME}Targets
  TARGETS lua luac FILE_SET HEADERS
  RUNTIME DESTINATION ${INSTALL_BINDIR} COMPONENT bin
  LIBRARY DESTINATION ${INSTALL_LIBDIR} COMPONENT lib
  ARCHIVE DESTINATION ${INSTALL_LIBDIR} COMPONENT lib
  PUBLIC_HEADER DESTINATION ${INSTALL_INCLUDEDIR} COMPONENT dev
)
install(
  EXPORT lib${PROJECT_NAME}Targets
  FILE lib${PROJECT_NAME}Targets.cmake
  DESTINATION ${INSTALL_CMAKEDIR}
)

# ------------------------------------------------------------------------------
# Create and install CMake configuration files for lua
include(CMakePackageConfigHelpers)
# https://cmake.org/cmake/help/latest/module/CMakePackageConfigHelpers.html
configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/cmake/lib${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/lib${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${INSTALL_CMAKEDIR}
  PATH_VARS INSTALL_DIR INSTALL_INCLUDEDIR INSTALL_LIBDIR
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/lib${PROJECT_NAME}ConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMinorVersion
)

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/lib${PROJECT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/lib${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION ${INSTALL_CMAKEDIR}
)
