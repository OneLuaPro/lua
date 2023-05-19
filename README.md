# Lua with CMake Support
Unaltered [Lua](http://www.lua.org/) source code and documentation with [CMake](https://cmake.org/) build infrastructure. This repository contains Lua Version 5.4.6 sources extracted from https://www.lua.org/ftp/lua-5.4.6.tar.gz. Tested generators and architectures are:

- Visual Studio 17 2022, Win32
- Visual Studio 17 2022, x64

**Notice:** Present `Makefiles` originate from Lua source code archive and are not used within the context of CMake-based build.

## Windows Tool Chain Preparation

A complete Microsoft Visual Studio Installation is optional but not strictly necessary. Simply install **Buildtools for Visual Studio 2022** from https://visualstudio.microsoft.com/de/downloads/#build-tools-for-visual-studio-2022 and select  the following minimum components for download and installation:

- MSVC v143 - VS 2022 C++-x64/x86-Buildtools
- C++-CMake-Tools for Windows
- Windows 11-SDK (10.0.22621.0) 

## Building and Installing Lua

Open `Developer Command Prompt for VS 2022` and change drive and directory. Download and unpack sources or simply clone this repository:

```cmd
c:
cd c:\Temp
git clone git@github.com:KritzelKratzel/lua.git
cd lua
```

CMake strongly encourages out-of-source builds.

```cmd
mkdir build
cd build
cmake .. -G "Visual Studio 17 2022" -A <arch>
cmake --build . --config Release
cmake --install . --config Release
```

Replace `<arch>` with your desired architecture. Available architectures with selected `Visual Studio 17 2022` generator are `Win32`, `x64`, `ARM` and `ARM64`. Default installation directory is `C:/Apps/lua-5.4.6` where a directory structure according to [GNU Coding Standards](https://www.gnu.org/prep/standards/html_node/Directory-Variables.html) is created:

- `bin`: Lua binaries `lua.exe` and `luac.exe`, both statically linked, no DLL dependency,
- `include`: Public header files,
- `lib`: Lua static library `liblua.lib`,
- `share`: documentation, man-pages, CMake configuration.

The default installation path can be overwritten by using `CMAKE_INSTALL_PREFIX` at the command line during CMake configuration. Example:

```cmd
cmake .. -G "Visual Studio 17 2022" -A x64 -DCMAKE_INSTALL_PREFIX="C:\Foo\Bar"
```

Finally add the path to Lua to Windows search path for executables. 

```cmd
setx PATH "%PATH%;C:\Apps\lua-5.4.6\bin"
```

Open an new command window and test lua. Use `CTRL-C` to leave Lua in interactive mode.

```cmd
C:\Users\John Doe>lua
Lua 5.4.6  Copyright (C) 1994-2023 Lua.org, PUC-Rio
> print(_VERSION)
Lua 5.4
>
C:\Users\John Doe>
```

## Additional Lua Modules

Table of Lua modules which smoothly fit into **Lua with CMake Support**:

| Extension Name                                               | Purpose                                                      | Forked from                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| [LuaSocket](https://github.com/KritzelKratzel/luasocket)     | Network support for the Lua language                         | https://github.com/lunarmodules/luasocket     |
| [librs232](https://github.com/KritzelKratzel/librs232)       | Multiplatform library for serial communications over RS-232 (serial port) | https://github.com/srdgame/librs232           |
| [LuaFileSystem](https://github.com/KritzelKratzel/luafilesystem) | Complements the set of functions related to file systems offered by the standard Lua distribution | https://github.com/lunarmodules/luafilesystem |

More will follow soon.

