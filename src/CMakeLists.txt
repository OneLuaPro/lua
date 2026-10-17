# MIT License
#
# Copyright (c) 1994–2026 Lua.org, PUC-Rio.
# Copyright (c) 2023-2026 The OneLuaPro project authors.
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
# Create the static Lua core library
add_library(lib${PROJECT_NAME}_objects OBJECT)
target_sources(lib${PROJECT_NAME}_objects
  PRIVATE lapi.c lctype.c lfunc.c lmathlib.c loslib.c ltable.c lundump.c
  lauxlib.c ldblib.c lgc.c lmem.c lparser.c ltablib.c lutf8lib.c lbaselib.c
  ldebug.c linit.c loadlib.c lstate.c ltm.c lvm.c lcode.c ldo.c liolib.c
  lobject.c lstring.c lzio.c lcorolib.c ldump.c llex.c lopcodes.c lstrlib.c
  lapi.h ldebug.h ljumptab.h lobject.h lprefix.h ltm.h lundump.h
  ldo.h llex.h lopcodes.h lstate.h  lvm.h lcode.h lfunc.h llimits.h lopnames.h
  lstring.h lzio.h lctype.h lgc.h lmem.h lparser.h ltable.h 
)
target_compile_definitions(lib${PROJECT_NAME}_objects PRIVATE LUA_BUILD_AS_DLL)
add_library(lib${PROJECT_NAME} SHARED $<TARGET_OBJECTS:lib${PROJECT_NAME}_objects>)
target_sources(lib${PROJECT_NAME}
  PUBLIC FILE_SET HEADERS
  FILES lua.h luaconf.h lualib.h lauxlib.h lua.hpp
)
target_compile_definitions(lib${PROJECT_NAME} PUBLIC LUA_BUILD_AS_DLL)
set_target_properties(lib${PROJECT_NAME} PROPERTIES
  WINDOWS_EXPORT_ALL_SYMBOLS TRUE	# export everything
  OUTPUT_NAME "lua"			# DLL-name: lua.dll
  PREFIX ""				# No prefix for DLL-name
  IMPORT_PREFIX "lib"			# Import-lib-name: liblua.lib
)
# Fix the output directory for the import library (.lib) during build
set_target_properties(lib${PROJECT_NAME} PROPERTIES
  ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
  LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
  RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)
target_include_directories(lib${PROJECT_NAME}
  PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
  "$<INSTALL_INTERFACE:${INSTALL_INCLUDEDIR}>"
)

# ------------------------------------------------------------------------------
# 2. Create the executables (Console and GUI versions). wlua prevents the
#    annoying CMD-Window in the background
add_executable(lua lua.c ${PROJECT_SOURCE_DIR}/logo/lua-logo-olp-dist.rc)
add_executable(luac luac.c $<TARGET_OBJECTS:lib${PROJECT_NAME}_objects>
  ${PROJECT_SOURCE_DIR}/logo/lua-logo-olp-dist.rc)
add_executable(wlua lua.c ${PROJECT_SOURCE_DIR}/logo/lua-logo-olp-dist.rc)
# Link all executables against the static core
target_link_libraries(lua PRIVATE lib${PROJECT_NAME})	# depends on lua.dll
target_link_libraries(luac PRIVATE)			# statically built
target_link_libraries(wlua PRIVATE lib${PROJECT_NAME})	# depends on lua.dll
if(WIN32)
  # Delay load of lua.dll, such that SetDllDirectoryW in lua.c has time to run
  # and use the modified DLL search order (LOAD_WITH_ALTERED_SEARCH_PATH). The
  # intention is to put lua.dll into lib/lua/5.5/ and not into bin/. This
  # requires a slight windows-only DLL search path extenstion in main() of lua.c.
  target_link_options(lua PRIVATE
    "LINKER:/DELAYLOAD:lua.dll"
    "LINKER:/DEPENDENTLOADFLAG:0x0"
  )
  target_link_options(wlua PRIVATE
    "LINKER:/DELAYLOAD:lua.dll"
    "LINKER:/DEPENDENTLOADFLAG:0x0"
  )
  # Link against delayimp.lib for the helper code in lua.c
  target_link_libraries(lua PRIVATE delayimp.lib)
  target_link_libraries(wlua PRIVATE delayimp.lib)
  # Configure wlua as a Windows GUI application (no console window)
  set_target_properties(wlua PROPERTIES WIN32_EXECUTABLE TRUE)
  # When using MSVC, the entry point must be explicitly set because
  # lua.c defines a standard "main" function instead of "WinMain":
  if(CMAKE_C_COMPILER_ID STREQUAL "IntelLLVM")
    target_link_options(wlua PRIVATE "/link" "/ENTRY:mainCRTStartup")
  elseif(MSVC)
    target_link_options(wlua PRIVATE "/ENTRY:mainCRTStartup")
  endif()
endif()

# ------------------------------------------------------------------------------
# 3. Installation instructions with Export-Sets
#    Export the shared core library
install(
  TARGETS lib${PROJECT_NAME}
  EXPORT lib${PROJECT_NAME}Targets
  RUNTIME DESTINATION ${INSTALL_TOP_CDIR} COMPONENT bin
  ARCHIVE DESTINATION ${INSTALL_LIBDIR} COMPONENT lib
  FILE_SET HEADERS DESTINATION ${INSTALL_INCLUDEDIR} COMPONENT dev
)
# Install both interpreters
install(
  TARGETS lua wlua
  RUNTIME DESTINATION ${INSTALL_BINDIR} COMPONENT bin
)
# Install compiler
install(
  TARGETS luac
  RUNTIME DESTINATION ${INSTALL_BINDIR} COMPONENT bin
)
# Generate the CMake configuration files for external projects
install(
  EXPORT lib${PROJECT_NAME}Targets
  FILE lib${PROJECT_NAME}Targets.cmake
  DESTINATION ${INSTALL_CMAKEDIR}
)
# Folder for user-supplied third-party runtime DLLs (e.g. Oracle Instant
# Client). It is put in front of PATH by lua.exe at startup (see lua.c).
# The README makes sure the folder exists and tells the user what it is for.
install(
  FILES ${PROJECT_SOURCE_DIR}/cmake/README-3rdparty.txt
  DESTINATION ${INSTALL_LIBDIR}/3rdparty
  RENAME README.txt
  COMPONENT bin
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
