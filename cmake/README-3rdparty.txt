OneLuaPro - third-party DLL folder (lib\3rdparty)
=================================================

This folder is intentionally (almost) empty.

lua.exe puts this folder in FRONT of PATH for its own process (and child
processes) at startup. DLLs placed here are found by all Lua modules, and
they take precedence over DLLs of the same name elsewhere on PATH. Nothing
is changed system-wide.

Use it for runtime libraries that OneLuaPro does not ship itself.


Oracle Instant Client (for luasql.oci8)
---------------------------------------

The Oracle driver luasql.oci8 needs the Oracle Instant Client runtime, which
is not part of OneLuaPro.

1. Download "Basic" or "Basic Light" (64-bit, version 23) from
   https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html
   and unpack the ZIP file.

2. Copy all DLL files (*.dll) from the unpacked folder into this folder.
   (Tested with Instant Client Basic Light 23.26. For that package, only
   four DLLs are needed for the plain driver: oci.dll, oraociicus.dll,
   orannz.dll and legacy.dll. Copying all of them avoids missing one and
   works for both packages.)

Alternatively, leave this folder empty and add an existing Instant Client
folder to PATH. Files in this folder win if both exist.


Requirements and troubleshooting
--------------------------------

- 64-bit only. The Instant Client must match the bitness of OneLuaPro (x64).
- Instant Client 23 does not support Windows 7.
- oci.dll needs the Microsoft Visual C++ runtime (VCRUNTIME140.dll). If it is
  missing, install the "Microsoft Visual C++ Redistributable" for x64.
- require "luasql.oci8" fails with "module not found" or with a message about
  oci.dll: the Instant Client files are not found. Check the file names and
  that this folder is "lib\3rdparty" next to "bin".
- Connecting fails with ORA-28041 or with an empty error message: some DLLs
  of the Instant Client are missing. Copy all *.dll files, not only oci.dll.


License
-------

The Oracle files are not part of OneLuaPro and are not covered by its
license. They are subject to the "Oracle Free Distribution, Hosting, and Use
Terms and Conditions" that come with the download (file *_LICENSE in the ZIP
file). If you pass the DLLs on to others, include that license file.
