---
layout: post
date: 2025-11-29
title: WebGL compatibility extension and DLL mysteries
categories: blog
tags: opengl webgl opencascade c++ opensource
permalink: /blog/2025-11-29-angle-webgl-compatibility/
thumbnail:
---

Recently, I've been pointed to the `EGL_ANGLE_create_context_webgl_compatibility` extension.
To play with it, I've replaced existing `libEGL.dll`/`libGLESv2.dll` alongside *OCCT* build with updated libraries from *ANGLE project*.
However, the process stuck at the very beginning - the new libraries worked just fine with my `wglinfo` tool, but not with *OCCT*.
Surprisingly, `eglGetDisplay()` returned `NULL` in *OCCT* builds. How might this mystery happen?

<!--break-->

*OCCT* supports *OpenGL ES* and on *Windows* platform could be linked to `libEGL`/`libGLESv2`
libraries provided by [ANGLE project](https://chromium.googlesource.com/angle/angle).
Desktop *OpenGL* is much more powerful and functional than *OpenGL ES*,
but *ANGLE project* allows testing compatibility with embedded systems (like *Android*).
Several *DRAW* test grids in *OCCT* share common tests, but run using different 3D graphic backends:

- `tests/opengl`
- `tests/opengles2`
- `tests/opengles3`

*ANGLE project* is actually used by browsers for bringing up *WebGL* support on *Windows* and some other platforms.
Although *WebGL* is defined on top of *OpenGL ES* specifications, it has more limitations and the strictest validation layer.
Passing *OpenGL ES 3* tests doesn't necessarily guarantee that the same code will work on *WebGL 2.0* (even when both implementations rely on *ANGLE project*).

Building OCCT as a *WebAssembly* would allow running *WebGL* tests directly in browser(s), but this is rather complex and slow.
And now we look at [`EGL_ANGLE_create_context_webgl_compatibility`](https://android.googlesource.com/platform/external/angle/+/HEAD/extensions/EGL_ANGLE_create_context_webgl_compatibility.txt)
extension, can we use it to verify *WebGL* compatibility using fast native OCCT builds?

## ANGLE enumerations

*ANGLE project* uses an unusual build system, tangled up dependencies, requires particular versions of build toolchains, etc.
If you are not *ANGLE project* developer, then you would clearly prefer to reuse some existing builds.
Unfortunately, the project itself doesn't provide any official releases nor binaries.

Legacy *OCCT development portal* provided *"Angle-gles2 2.1.0"* binaries (`ANGLE 2.1.0.46ad513f4e5b`),
which haven't `EGL_ANGLE_create_context_webgl_compatibility` extension.
But then I've realized that the DLLs `ANGLE 2.1.0.57ea533f79a7` on my local system are actually different, and support this extension.
Both libraries confusingly report the same `ANGLE 2.1.0` version,
but apparently they are separated by several years of intensive development if you'll compare hash codes in the source tree!

Anyway, lucky me, my local *ANGLE* DLLs should support this extension - lets go ahead and try it!

```cpp
  EGLint anEglCtxAttribs3[] = {
    EGL_CONTEXT_CLIENT_VERSION, 3,
    // EGL_CONTEXT_WEBGL_COMPATIBILITY_ANGLE
    0x33AC, EGL_TRUE,
    EGL_NONE, EGL_NONE
  };
  myEglContext = eglCreateContext((EGLDisplay )myEglDisplay, myEglConfig, EGL_NO_CONTEXT, anEglCtxAttribs3);
```

Nope, something's wrong: `eglCreateContext()` returns `NULL` and `eglGetError()` returns `EGL_BAD_ATTRIBUTE`... But why?
After fruitless trials with the code, I've downloaded *ANGLE* git repository and found the following interesting commit:

```diff
Author: Jamie Madill <jmadill@chromium.org>  2018-09-13 18:20:52

    Fix EGL enum allocation.
...
--------- extensions/EGL_ANGLE_create_context_webgl_compatibility.txt ---------
index 2e7fb825d6..4245d6ca73 100644
@@ -53,7 +53,7 @@ New Tokens
     Accepted as an attribute name in the <*attrib_list> argument to
     eglCreateContext:

-        EGL_CONTEXT_WEBGL_COMPATIBILITY_ANGLE 0x3AAC
+        EGL_CONTEXT_WEBGL_COMPATIBILITY_ANGLE 0x33AC

 Additions to the EGL 1.4 Specification
```

Gotcha! So it should be just a *misprint* in *ANGLE* source code...
Temporarily changing the enumeration value in the code fixed this strange error.

But wait! This misprint was fixed in *__2018__*, seven years ago!
That means that *ANGLE* libraries in my local environment were extremely old.
May I find a more decent build?

## Looking for a decent ANGLE

[*MSYS2*](https://www.msys2.org/) package manager provides
package [`mingw-w64-angleproject`](https://packages.msys2.org/base/mingw-w64-angleproject)
with four binary builds available for `ucrt64`, `mingw64`, `clang64`, and `clangarm64`.
It's better to download them using *MSYS2* itself, as links in web-interface
would require manually downloading all dependencies (`libc++`, `zlib1` and others):

```
$ pacman -S mingw-w64-clang-x86_64-angleproject
resolving dependencies...
looking for conflicting packages...

Packages (10) mingw-w64-clang-x86_64-egl-headers-1.5.r284.3ae2b7c-1
              mingw-w64-clang-x86_64-gles-headers-3.2.r1065.7fc154c-1
              mingw-w64-clang-x86_64-jsoncpp-1.9.6-3
              mingw-w64-clang-x86_64-libc++-20.1.8-1
              mingw-w64-clang-x86_64-libjpeg-turbo-3.1.1-1
              mingw-w64-clang-x86_64-libpng-1.6.50-1
              mingw-w64-clang-x86_64-libunwind-20.1.8-1
              mingw-w64-clang-x86_64-rapidjson-1.1.0-8
              mingw-w64-clang-x86_64-zlib-1.3.1-1
              mingw-w64-clang-x86_64-angleproject-2.1.r25748.890b5d8f-1
```

`wglinfo` loaded libraries just fine:
```
[EGL] EGLVersion:    1.5 (ANGLE 2.1.25748 git hash: 890b5d8fa298)
```

_Instead of MSYS2 may consider taking DLLs from Browser installation - like Firefox;_
_although browsers usually use customized builds of ANGLE project that might bring additional issues._

I've replaced DLLs and started `DRAWEXE` - only to see the mysterious error:

```
pload VISUALIZATION
vdriver -load GLES
vinit

> Error: no EGL display
```

The C++ code, trivial as could possible be, *__FAILED__* for no clear reason:

```cpp
  myEglDisplay = eglGetDisplay(EGL_DEFAULT_DISPLAY);
  if ((EGLDisplay )myEglDisplay == EGL_NO_DISPLAY)
  {
    ::Message::SendFail("Error: no EGL display");
    return false;
  }
```

How can this call ever fail, if the code just requests a default EGL display?
Moreover, why `wglinfo` works with these DLLs and `DRAWEXE` doesn't?

## Import lookup tables

I've compared file headers, enumeration values, [manifest files](https://github.com/gkv311/wglinfo/issues/2)
embedded into DLLs and was almost ready to give up finding a rational explanation.
The only difference between OCCT and `wglinfo` was that OCCT linked to `libEGL.dll` at build time,
while `wglinfo` loaded DLL and its functions dynamically via `LoadLibraryW()` and `GetProcAddress()`.

Then I've started to dig into `TKOpenGles.dll` and found peculiar section:

```
 libEGL.dll
	Import Lookup Table RVA:  00253710h (Unbound IAT)
	TimeDateStamp:            00000000h
	ForwarderChain:           00000000h
	DLL Name RVA:             0025966Eh
	Import Address Table RVA: 001A0EB8h
	First thunk RVA:          001A0EB8h
	Ordn  	Name
	-----	-----
	  28
	  34
	  25
	   5
	  32
	  22
	  26
...
   KERNEL32.dll
	Import Lookup Table RVA:  00252858h (Unbound IAT)
	TimeDateStamp:            00000000h
	ForwarderChain:           00000000h
	DLL Name RVA:             0025A7DAh
	Import Address Table RVA: 001A0000h
	First thunk RVA:          001A0000h
	Ordn  	Name
	-----	-----
	1459 	VirtualQuery
	 681 	GetProcessHeap
	 828 	HeapFree
	 824 	HeapAlloc
```

All functions from `libEGL.dll` and `libGLESv2.dll` were imported by *ordinal numbers* and not their names!
Loading functions using fixed offsets in DLL instead of name lookup is intended to speedup loading large libraries.
I've heard about [this optimization](https://learn.microsoft.com/en-us/cpp/build/exporting-functions-from-a-dll-by-ordinal-rather-than-by-name) before,
but never seen it to be used in modern practices as it has considerable side effect.
Changing the order of exported functions in DLL and other changes would make ordinals incorrect.

Apparently, the problem was hidden inside `libEGL.lib` taken from *OCCT Development Portal* and used for building OCCT.
*MSYS2* builds don't provide `*.lib` files (only `*.a` files),
and I don't know if a fresh *ANGLE* built using *Visual Studio* would have the same issue or it was fixed at some point.

Instead of tangling *ANGLE project* itself, I've generated my own `*.lib` files
using the same old method as I've used for [linking *sView* with *FFmpeg* libraries](https://github.com/gkv311/sview-deps-wnt-ffmpeg2lib).
I've created a [dummy project](https://github.com/gkv311/wglinfo/tree/HEAD/libegldummy) `libEGL` that listed all exported functions with an empty body:

```cpp
// libEGL.cpp
#include <EGL/eglplatform.h>

#undef  EGLAPI
#define EGLAPI __declspec(dllexport)
#define EGLBODY0 {return 0;}

extern "C" {
EGLAPI EGLDisplay EGLAPIENTRY eglGetCurrentDisplay (void) EGLBODY0;
EGLAPI EGLSurface EGLAPIENTRY eglGetCurrentSurface (EGLint readdraw) EGLBODY0;
EGLAPI EGLDisplay EGLAPIENTRY eglGetDisplay (EGLNativeDisplayType display_id) EGLBODY0;
EGLAPI EGLint EGLAPIENTRY eglGetError (void) EGLBODY0;
...
}
```

After rebuilding `TKOpenGles.dll` linked via newly generated `libEGL.lib`/`libGLESv2.lib`,
I've rechecked image file header and found function names in place:

```
   libEGL.dll
	Import Lookup Table RVA:  00253720h (Unbound IAT)
	TimeDateStamp:            00000000h
	ForwarderChain:           00000000h
	DLL Name RVA:             00259802h
	Import Address Table RVA: 001A0EB8h
	First thunk RVA:          001A0EB8h
	Ordn  	Name
	-----	-----
	  21 	eglGetCurrentDisplay
	  26 	eglGetProcAddress
	  29 	eglMakeCurrent
	  32 	eglQueryString
...
```

Running `DRAWEXE` with replaced *ANGLE* DLLs finally succeeded!

## Testing WebGL compatibility

The fast testing of `tests/opengles3` grid with `EGL_ANGLE_create_context_webgl_compatibility` extension enabled
shows somewhat about 31 tests failed:

```
Failed:
  opengles3 general msaa
  opengles3 geom interior1
  opengles3 geom interior2
  opengles3 text bug25732_2
  opengles3 textures alpha_mask
  ...
```

The tests are failed due to lack of features supported by *OpenGL ES 3.2*,
but not by *WebGL 2.0*, which is expected result:

```
TKOpenGl | Type: Error | ID: 0 | Severity: High | Message:
  Failed to compile Fragment Shader [occt_phong-l_d-00c0]. Compilation log:
  ERROR: 0:497: 'aPlaneIter' : Loop index cannot be compared with non-constant expression
...
TKOpenGl | Type: Error | ID: 0 | Severity: High | Message:
  OpenGL context does not support floating-point RGBA color buffer format.
...
TKOpenGl | Type: Error | ID: 0 | Severity: High | Message:
  Error: No suitable texture format for BGRA image format [Graphic3d_MarkerImage_1323]
```

The list of supported *OpenGL ES* extensions dropped from `109` to a somewhat very basic `23` after enabling this extension:

```
[EGL] OpenGL ES extensions:
    GL_AMD_performance_monitor, GL_ANGLE_client_arrays, GL_ANGLE_depth_texture,
    GL_ANGLE_get_serialized_context_string, GL_ANGLE_program_cache_control,
    GL_ANGLE_request_extension, GL_ANGLE_robust_client_memory,
    GL_ANGLE_translated_shader_source, GL_ANGLE_webgl_compatibility,
    GL_CHROMIUM_bind_generates_resource, GL_CHROMIUM_bind_uniform_location,
    GL_CHROMIUM_copy_compressed_texture, GL_CHROMIUM_copy_texture,
    GL_EXT_debug_label, GL_EXT_debug_marker, GL_EXT_discard_framebuffer,
    GL_EXT_robustness, GL_KHR_debug, GL_NV_fence, GL_OES_depth24,
    GL_OES_depth32, GL_OES_packed_depth_stencil, GL_OES_surfaceless_context.
```

Now, with this extension at hand, it is possible to perform preliminary *WebGL 2.0* compatibility tests
without building *WebAssembly* (well, at least to some degree, as actual behavior in different browsers might vary).
