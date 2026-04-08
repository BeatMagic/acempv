# acempv

Prebuilt [mpv](https://mpv.io/) player library for Windows.

## Build Origin

The prebuilt `libmpv-2.dll` is a **MinGW-compiled** binary obtained from upstream
mpv Windows builds. Identifying traits:

- `lib` prefix on the DLL name (MinGW convention; MSVC would produce `mpv-2.dll`)
- `mpv.lib` is a thin import library generated from `mpv.def`, not a native MSVC
  build artifact
- The DLL statically links all codec dependencies (libav, etc.), hence the large
  file size

## ABI Boundary

The library exposes a **pure C API** (`extern "C"`, opaque handles, plain C types).
No C++ objects, exceptions, or CRT state cross the DLL boundary. Memory ownership
is explicit — mpv-allocated strings must be freed with `mpv_free()`.

This means:

- **Safe to link from MSVC** despite the DLL being MinGW-compiled — the C ABI is
  stable across compilers on Windows x64.
- **Safe to use in any build configuration** (Debug, Release, RelWithDebInfo) —
  there is no debug/release CRT mismatch because no CRT objects are shared across
  the boundary.
