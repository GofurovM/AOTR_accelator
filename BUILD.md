# Building

- Windows, **Visual Studio Build Tools** (2019 or 2022) with "Desktop development with C++" (the x86 compiler).
  `build.bat` finds `vcvars32.bat` through vswhere; set `AOTR_VCVARS` to its full path if it is somewhere else.
- `build.bat` - release build -> `out\bfme2_accel.dll` (with its version resource from `src\bfme2_accel.rc`), `out\aotr_start.exe` (with the ring icons from `res\`, compiled by `rc`) and `out\bfme2_accel.map`.
- `PLAY.bat` - build, then start AOTR through `out\aotr_start.exe` (administrator rights are asked for).
- `PACKAGE.bat` - build and make `AOTR_accelerator.zip` (DLL + `AOTR_ACCELERATOR.exe` + the guides from `guide\`) for players; one click installs it.

The DLL is one translation unit: `src/aotr_accel.cpp` includes every `src/aotr_*.inc`. `vendor/rpmalloc` is
compiled and linked but not used as the allocator (the game's own heap is kept).
