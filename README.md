# Lua with CMake Support
Almost unaltered [Lua](http://www.lua.org/) source code and documentation with [CMake](https://cmake.org/) build infrastructure. This repository contains different Lua versions extracted from https://www.lua.org/download.html. Tested generators and architectures are:

- Visual Studio 17 2022, Win32
- Visual Studio 17 2022, x64

**Notice:** Present `Makefiles` originate from Lua source code archive and are not used within the context of CMake-based build.

## Windows Tool Chain Preparation

A complete Microsoft Visual Studio Installation is optional but not strictly necessary. Simply install **Buildtools for Visual Studio 2022** from https://visualstudio.microsoft.com/de/downloads/#build-tools-for-visual-studio-2022 and select  the following minimum components for download and installation:

- MSVC v143 - VS 2022 C++-x64/x86-Buildtools
- C++-CMake-Tools for Windows
- Windows 11-SDK

## Building and Installing Lua

Open `Developer Command Prompt for VS 2022` and change drive and directory. Download and unpack sources or simply clone this repository:

```cmd
c:
cd c:\Temp
git clone https://github.com/OneLuaPro/lua.git
cd lua
```

CMake strongly encourages out-of-source builds.

```cmd
mkdir build && cd build
cmake .. -G "Visual Studio 17 2022" -A <arch>
cmake --build . --config Release
cmake --install . --config Release
```

Replace `<arch>` with your desired architecture. Available architectures with selected `Visual Studio 17 2022` generator are `Win32`, `x64`, `ARM` and `ARM64`. Default installation directory is `C:\Apps\lua-<VERSION>` where a directory structure according to [GNU Coding Standards](https://www.gnu.org/prep/standards/html_node/Directory-Variables.html) is created:

- `bin`: Lua binaries `lua.exe`, `wlua.exe` (see below) and `luac.exe`, all statically linked, no DLL dependency besides to `lua.dll`,
- `include`: Public header files,
- `lib`: Lua static library `liblua.lib`,
- `share`: documentation, man-pages, CMake configuration.

The default installation path can be overwritten by using `CMAKE_INSTALL_PREFIX` at the command line during CMake configuration. Example:

```cmd
cmake .. -G "Visual Studio 17 2022" -A x64 -DCMAKE_INSTALL_PREFIX="C:\Foo\Bar"
```

Finally add the path to Lua to Windows search path for executables. 

```cmd
setx PATH "%PATH%;<INSTALL_PREFIX>\lua-<VERSION>\bin"
```

Open an new command window and test lua. Use `CTRL-C` to leave Lua in interactive mode.

```cmd
C:\Users\John Doe>lua
Lua 5.5.0  Copyright (C) 1994-2025 Lua.org, PUC-Rio
> print(_VERSION)
Lua 5.5
>
C:\Users\John Doe>
```

## Extras

In addition to the standard Lua executables `lua.exe` and `luac.exe`, a third binary is built: `wlua.exe`. This executable is functionally identical to `lua.exe` but is compiled for a different **Windows subsystem** (Windows GUI instead of Console). This is particularly useful for Lua-based GUI applications, as it prevents the annoying empty terminal window from appearing in the background. Lua GUI applications can be launched using a batch file as follows:

```bat
@echo off
pushd "%~dp0"
start "" wlua.exe "%~dp0prog.lua" %*
popd
exit
```

## Upgrading to a New Lua Version

This is a brief guide on how to integrate new vanilla Lua releases (provided as tarballs) into this codebase. It is recommended to perform all steps using the Git Bash terminal.

```bash
# 1. Download and unpack the current and new Lua tarballs in the directory where your local repo resides:
$ ls -l
drwxr-xr-x 1 John Doe 197121      0 Aug 29 08:04 lua/
drwxr-xr-x 1 John Doe 197121      0 Dec 15  2025 lua-5.5.0/
drwxr-xr-x 1 John Doe 197121      0 Jul 24 16:07 lua-5.5.1/

# 2. Create a vanilla diff from the current version to the new Lua version:
$ diff -urN lua-5.5.0 lua-5.5.1 > lua-550-551.patch

# 3. Change into your local Git repository:
$ cd lua

# 4. Perform a dry-run with the patch command. Some failed hunks may occur due to local customizations:
$ patch -p1 < ../lua-550-551.patch --dry-run

# 5. Apply the actual patch to the codebase:
$ patch -p1 < ../lua-550-551.patch

# 6. Locate any rejected patch files (.rej):
$ find . -name "*.rej"
./src/lua.h.rej

# 7. Manually resolve the conflicts shown in the .rej files using an editor, then delete the .rej files.

# 8. Clean up all temporary backup (*.orig) files created by the patch utility:
$ find . -name "*.orig" -delete

# 9. Check for empty files left behind by deleted upstream files, and remove them if any exist:
$ find . -type f -empty -delete

# 10. Perform a test compilation, then stage and commit the changes:
$ git add .
$ git commit -m "Upgrade Lua core to v5.5.1"

# 11. Tag the new release version:
$ git tag -a v5.5.1-0 -m "Lua v5.5.1-0 for OneLuaPro"

# 12. Push the commits and the new tag to the remote repository:
$ git push origin main
$ git push origin v5.5.1-0
```

## License

See `https://github.com/OneLuaPro/lua/blob/main/LICENSE`.
