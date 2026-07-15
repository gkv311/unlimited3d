---
layout: post
date: 2026-07-15
title: Visuals, Visuals, Visuals!
categories: tools
tags: opengl opensource tools
permalink: /tools/2026-07-15-visuals/
thumbnail:
---

Developers that tried to create a window with an OpenGL context on their own have to pass the color+depth+stencil buffers bitness
to methods like `eglChooseConfig()`/`ChoosePixelFormat()`/`glXChooseFBConfig()`,
which is most likely a 32-bit RGBA + 24-bit depth + 8-bit stencil configuration.

But if you take a deeper look at `DescribePixelFormat()`/`glXGetFBConfigAttrib()`,
you'll have a chance to briefly touch on the long history of computer graphics.

Recently I've spent some time on improving [wglinfo](https://github.com/gkv311/wglinfo/releases)
and added a listing of visuals using `GLX` (Linux) and `CGL` (macOS).

<!--break-->


## Querying visuals

The *visual*, or *Pixel Format Descriptor*, or *Frame-buffer config*, defines properties of a window buffers' chain suitable for OpenGL rendering.
The APIs for incorporating *visuals* are platform-specific: there is `WGL` for *Windows*, `GLX` for *X11* (*Linux* and others),
`CGL` on *macOS* and `EGL` for newer systems (*Android*, *Wayland* on *Linux*, etc.).

Lets first briefly take a look at these APIs.

### WGL pixel format descriptors

On *Windows* (`WGL`) platform, the function `ChoosePixelFormat()` finds the pixel format matching or closest to the requested one.
There is also an extension `WGL_ARB_pixel_format` and method `wglChoosePixelFormatARB()` with more capabilities.

The function `DescribePixelFormat()` is an entry point for querying information about visuals.
When called with zeros in arguments it returns the number of available visuals;
when called with a valid visual index (starting from `1`), it fills in the structure `PIXELFORMATDESCRIPTOR`:

```cpp
  HWND aWindow = ...;
  HDC  aDevCtx = ::GetDC(aWindow);

  const int aNbFormats = DescribePixelFormat(aDevCtx, 0, 0, nullptr);
  for (int aFormatIter = 1; aFormatIter <= aNbFormats; ++aFormatIter)
  {
    PIXELFORMATDESCRIPTOR aFormat = {};
    aFormat.nSize = (WORD)sizeof(PIXELFORMATDESCRIPTOR);
    DescribePixelFormat(aDevCtx, aFormatIter, aFormat.nSize, &aFormat);
    if ((aFormat.dwFlags & PFD_SUPPORT_OPENGL) == 0)
      continue;

    ...
  }
```

The function `wglGetPixelFormatAttribivARB()` from `WGL_ARB_pixel_format` extension might be used to query more attributes,
like multi-sampling configuration provided by `WGL_ARB_multisample` extension.

On *Windows* platform, hardware-accelerated OpenGL implementation are provided by *ICD* (Installable Client Driver).
Before that, there was also *MCD* (Mini-Client Driver) interface, which is now obsolete.

When there is no OpenGL driver available in system, *Windows* provides a very outdated
software implementation of *OpenGL 1.1*, suitable almost for nothing nowadays.
Here is a list of `WGL` visuals without *ICD* (OpenGL Installable Client Driver) on Windows:

<details><summary>WGL visuals</summary>
<pre>
[WGL] 36 WGL Visuals
      visual  bf lv rg d st  colorbuffer  sr ax dp st accumbuffer msaa  cav
  id  dep cl  sz l  ci b ro  r  g  b  a F gb bf th cl  r  g  b  a ns  b eat
------------------------------------------------------------------------
    1  24 wb  32  . r  . .   8  8  8  0 .  .  . 32  8 16 16 16  .  0  0 Software
    2  24 wb  32  . r  . .   8  8  8  0 .  .  . 16  8 16 16 16  .  0  0 Software
    3  24 wn  32  . r  y .   8  8  8  0 .  .  . 32  8 16 16 16  .  0  0 Software
    4  24 wn  32  . r  y .   8  8  8  0 .  .  . 16  8 16 16 16  .  0  0 Software
    5  24 wb  32  . r  . .   8  8  8  8 .  .  . 32  8 16 16 16 16  0  0 Software
    6  24 wb  32  . r  . .   8  8  8  8 .  .  . 16  8 16 16 16 16  0  0 Software
    7  24 wn  32  . r  y .   8  8  8  8 .  .  . 32  8 16 16 16 16  0  0 Software
    8  24 wn  32  . r  y .   8  8  8  8 .  .  . 16  8 16 16 16 16  0  0 Software
    9   0 wb  32  . i  . .   .  .  .  . .  .  . 32  8  .  .  .  .  0  0 Software
   10   0 wb  32  . i  . .   .  .  .  . .  .  . 16  8  .  .  .  .  0  0 Software
   11   0 wn  32  . i  y .   .  .  .  . .  .  . 32  8  .  .  .  .  0  0 Software
   12   0 wn  32  . i  y .   .  .  .  . .  .  . 16  8  .  .  .  .  0  0 Software
   13  24 bm  24  . r  . .   8  8  8  0 .  .  . 32  8 16 16 16  .  0  0 Software
   14  24 bm  24  . r  . .   8  8  8  0 .  .  . 16  8 16 16 16  .  0  0 Software
   15  24 bm  24  . r  . .   8  8  8  8 .  .  . 32  8 16 16 16 16  0  0 Software
   16  24 bm  24  . r  . .   8  8  8  8 .  .  . 16  8 16 16 16 16  0  0 Software
   17   0 bm  24  . i  . .   .  .  .  . .  .  . 32  8  .  .  .  .  0  0 Software
   18   0 bm  24  . i  . .   .  .  .  . .  .  . 16  8  .  .  .  .  0  0 Software
   19  15 bm  16  . r  . .   5  5  5  0 .  .  . 32  8 11 11 10  .  0  0 Software
   20  15 bm  16  . r  . .   5  5  5  0 .  .  . 16  8 11 11 10  .  0  0 Software
   21  15 bm  16  . r  . .   5  5  5  8 .  .  . 32  8  8  8  8  8  0  0 Software
   22  15 bm  16  . r  . .   5  5  5  8 .  .  . 16  8  8  8  8  8  0  0 Software
   23   0 bm  16  . i  . .   .  .  .  . .  .  . 32  8  .  .  .  .  0  0 Software
   24   0 bm  16  . i  . .   .  .  .  . .  .  . 16  8  .  .  .  .  0  0 Software
   25   8 bm   8  . r  . .   3  3  2  0 .  .  . 32  8 11 11 10  .  0  0 Software
   26   8 bm   8  . r  . .   3  3  2  0 .  .  . 16  8 11 11 10  .  0  0 Software
   27   8 bm   8  . r  . .   3  3  2  8 .  .  . 32  8  8  8  8  8  0  0 Software
   28   8 bm   8  . r  . .   3  3  2  8 .  .  . 16  8  8  8  8  8  0  0 Software
   29   0 bm   8  . i  . .   .  .  .  . .  .  . 32  8  .  .  .  .  0  0 Software
   30   0 bm   8  . i  . .   .  .  .  . .  .  . 16  8  .  .  .  .  0  0 Software
   31   3 bm   4  . r  . .   1  1  1  0 .  .  . 32  8  5  6  5  .  0  0 Software
   32   3 bm   4  . r  . .   1  1  1  0 .  .  . 16  8  5  6  5  .  0  0 Software
   33   3 bm   4  . r  . .   1  1  1  8 .  .  . 32  8  4  4  4  4  0  0 Software
   34   3 bm   4  . r  . .   1  1  1  8 .  .  . 16  8  4  4  4  4  0  0 Software
   35   0 bm   4  . i  . .   .  .  .  . .  .  . 32  8  .  .  .  .  0  0 Software
   36   0 bm   4  . i  . .   .  .  .  . .  .  . 16  8  .  .  .  .  0  0 Software
------------------------------------------------------------------------
  id  dep cl  bf lv rg d st  r  g  b  a F sr ax dp st  r  g  b  a ns  b cav
      visual  sz l  ci b ro  colorbuffer  gb bf th cl accumbuffer msaa  eat
------------------------------------------------------------------------
</pre>
</details>

With an OpenGL driver installed in the system, `WGL` will return visuals supported (or emulated) by a particular graphics hardware and driver.

The extension `WGL_ARB_pixel_format` further expands the list with so-called 'non-displayable' pixel formats,
which cannot be set to a window, but can be used for offscreen buffers created by `wglCreatePbufferARB()` and similar methods.
These, however, should be rather replaced by creating *Frame Buffer Object* in most cases.

Here is a sample listing of extra formats returned by `WGL_ARB_pixel_format`:

<details><summary>WGL extra visuals</summary>
<pre>
[WGL] 256 WGL Visuals (96 basic + 160 extra)
      visual  bf lv rg d st  colorbuffer  sr ax dp st accumbuffer msaa  cav
  id  dep cl  sz l  ci b ro  r  g  b  a F gb bf th cl  r  g  b  a ns  b eat
------------------------------------------------------------------------
    1  24 wn  32  . r  . .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
    2  24 wb  32  . r  . .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 Software
    3  24 wn  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
...
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   97  30  .  32  . r  . .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
   98  30  .  32  . r  y .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
   99  30  .  32  . r  . .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
  100  30  .  32  . r  y .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
  101  30  .  32  . r  . .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
  102  30  .  32  . r  y .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
  103  30  .  32  . r  . .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
  104  30  .  32  . r  y .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
  105  30  .  30  . r  . .  10 10 10  0 .  .  . 24  8  .  .  .  .  0  0 None
  106  30  .  30  . r  y .  10 10 10  0 .  .  . 24  8  .  .  .  .  0  0 None
  107  30  .  30  . r  . .  10 10 10  0 .  .  . 24  0  .  .  .  .  0  0 None
  108  30  .  30  . r  y .  10 10 10  0 .  .  . 24  0  .  .  .  .  0  0 None
  109  30  .  30  . r  . .  10 10 10  0 .  .  . 16  0  .  .  .  .  0  0 None
  110  30  .  30  . r  y .  10 10 10  0 .  .  . 16  0  .  .  .  .  0  0 None
  111  30  .  30  . r  . .  10 10 10  0 .  .  .  0  0  .  .  .  .  0  0 None
  112  30  .  30  . r  y .  10 10 10  0 .  .  .  0  0  .  .  .  .  0  0 None
  113  30  .  32  . r  . .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
  114  30  .  32  . r  y .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
  115  30  .  32  . r  . .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
  116  30  .  32  . r  y .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
  117  30  .  32  . r  . .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
  118  30  .  32  . r  y .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
  119  30  .  32  . r  . .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
  120  30  .  32  . r  y .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
  121  30  .  30  . r  . .  10 10 10  0 .  .  . 24  8  .  .  .  .  0  0 None
...
------------------------------------------------------------------------
  id  dep cl  bf lv rg d st  r  g  b  a F sr ax dp st  r  g  b  a ns  b cav
      visual  sz l  ci b ro  colorbuffer  gb bf th cl accumbuffer msaa  eat
------------------------------------------------------------------------
</pre>
</details>


### GLX frame-buffer configs

*Linux* and other *Unix* systems relying on *X11* server, defines `GLX` API for managing OpenGL contexts.
`GLX 1.3+` provides functions like `glXChooseFBConfig()` to find suitable frame-buffer config,
and functions `glXGetFBConfigs()` + `glXGetFBConfigAttrib()` to query `GLXFB` configs from `Display`:

```cpp
  Display*  aDisp    = ...;
  const int aScreen  = DefaultScreen(aDisp);

  int          aFBCount = 0;
  GLXFBConfig* aFBCfgList = glXGetFBConfigs(aDisp, aScreen, &aFBCount);
  for (int aConfigIter = 0; aConfigIter < aFBCount; ++aConfigIter)
  {
    const GLXFBConfig anFBConfig = aFBCfgList[aConfigIter];
    int anFBConfigId = 0, aBufferSize = 0;
    glXGetFBConfigAttrib(aDisp, anFBConfig, GLX_FBCONFIG_ID, &anFBConfigId);
    glXGetFBConfigAttrib(aDisp, anFBConfig, GLX_BUFFER_SIZE, &aBufferSize);
    ...
  }
  XFree(aFBCfgList);
```

Prior to `GLX 1.3` there was another API for choosing `XVisualInfo` instead of `GLXFBConfig`,
but at some point this structure was found incomplete for OpenGL setup.

Apart from that, the listing of `GLXGB` configs looks pretty same as for `WGL` pixel format descriptors.

On Linux, OpenGL stack is normally provided by [Mesa 3D](https://mesa3d.org/) project,
with both hardware-accelerated and software-emulated implementations supporting the latest OpenGL versions.
Some systems may also use proprietary OpenGL drivers, though.

Here is a sample listing from `Xvfb` (a virtual offscreen server useful for testing):

<details><summary>GLXFB Configs</summary>
<pre>
[GLX] 240 GLXFB Configs
      visual  bf lv rg d st  colorbuffer  sr ax dp st accumbuffer msaa  cav
  id  dep cl  sz l  ci b ro  r  g  b  a F gb bf th cl  r  g  b  a ns  b eat
------------------------------------------------------------------------
0x043  24 wb  32  . r  . .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 Software
0x044  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 Software
0x045  24 wb  32  . r  . .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 Software
0x046  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 Software
0x047  24 wb  32  . r  . .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 Software
0x048  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 Software
0x049  24 wb  32  . r  . .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 Software
0x04a  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 Software
0x04d  24 wb  24  . r  . .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 Software
0x04e  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 Software
0x04f  24 wb  24  . r  . .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 Software
0x050  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 Software
0x051  24 wb  24  . r  . .   8  8  8  0 .  .  . 16  0  .  .  .  .  0  0 Software
0x052  24 wb  24  . r  y .   8  8  8  0 .  .  . 16  0  .  .  .  .  0  0 Software
0x053  24 wb  24  . r  . .   8  8  8  0 .  .  .  0  0  .  .  .  .  0  0 Software
0x054  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  0  .  .  .  .  0  0 Software
0x057  24 wb  32  . r  . .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 Software
0x058  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 Software
0x059  24 wb  32  . r  . .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 Software
0x05a  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 Software
0x05b  24 wb  32  . r  . .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 Software
0x05c  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 Software
0x05d  24 wb  32  . r  . .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 Software
0x05e  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 Software
0x061  24 wb  24  . r  . .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 Software
0x062  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 Software
0x063  24 wb  24  . r  . .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 Software
0x064  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 Software
0x065  24 wb  24  . r  . .   8  8  8  0 .  .  . 16  0  .  .  .  .  0  0 Software
...
0x152  15 bm  16  . r  y .   5  5  5  1 .  .  . 24  8  .  .  .  .  0  0 Software
0x153  15 bm  16  . r  . .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 Software
0x154  15 bm  16  . r  y .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 Software
0x155  15 bm  16  . r  . .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 Software
0x156  15 bm  16  . r  y .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 Software
0x157  15 bm  16  . r  . .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 Software
0x158  15 bm  16  . r  y .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 Software
0x165  12 bm  16  . r  . .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 Software
0x166  12 bm  16  . r  y .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 Software
0x167  12 bm  16  . r  . .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 Software
0x168  12 bm  16  . r  y .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 Software
0x169  12 bm  16  . r  . .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 Software
0x16a  12 bm  16  . r  y .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 Software
0x16b  12 bm  16  . r  . .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 Software
0x16c  12 bm  16  . r  y .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 Software
0x179  16 bm  16  . r  . .   5  6  5  0 .  .  . 24  8  .  .  .  .  0  0 Software
0x17a  16 bm  16  . r  y .   5  6  5  0 .  .  . 24  8  .  .  .  .  0  0 Software
0x17b  16 bm  16  . r  . .   5  6  5  0 .  .  . 24  0  .  .  .  .  0  0 Software
0x17c  16 bm  16  . r  y .   5  6  5  0 .  .  . 24  0  .  .  .  .  0  0 Software
0x17d  16 bm  16  . r  . .   5  6  5  0 .  .  . 16  0  .  .  .  .  0  0 Software
0x17e  16 bm  16  . r  y .   5  6  5  0 .  .  . 16  0  .  .  .  .  0  0 Software
0x17f  16 bm  16  . r  . .   5  6  5  0 .  .  .  0  0  .  .  .  .  0  0 Software
0x180  16 bm  16  . r  y .   5  6  5  0 .  .  .  0  0  .  .  .  .  0  0 Software
0x183  15 bm  16  . r  . .   5  5  5  1 .  .  . 24  8  .  .  .  .  0  0 Software
0x184  15 bm  16  . r  y .   5  5  5  1 .  .  . 24  8  .  .  .  .  0  0 Software
0x185  15 bm  16  . r  . .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 Software
0x186  15 bm  16  . r  y .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 Software
0x187  15 bm  16  . r  . .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 Software
0x188  15 bm  16  . r  y .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 Software
0x189  15 bm  16  . r  . .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 Software
0x18a  15 bm  16  . r  y .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 Software
0x197  12 bm  16  . r  . .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 Software
0x198  12 bm  16  . r  y .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 Software
0x199  12 bm  16  . r  . .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 Software
0x19a  12 bm  16  . r  y .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 Software
0x19b  12 bm  16  . r  . .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 Software
0x19c  12 bm  16  . r  y .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 Software
0x19d  12 bm  16  . r  . .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 Software
0x19e  12 bm  16  . r  y .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 Software
0x1ab  24 wb  32  . r  . .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 Software
0x1ac  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 Software
0x1ad  24 wb  32  . r  . .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 Software
0x1ae  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 Software
0x1af  24 wb  32  . r  . .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 Software
0x1b0  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 Software
0x1b1  24 wb  32  . r  . .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 Software
0x1b2  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 Software
0x1b5  24 wb  32  . r  . .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 Software
0x1b6  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 Software
0x1b7  24 wb  32  . r  . .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 Software
0x1b8  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 Software
0x1b9  24 wb  32  . r  . .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 Software
0x1ba  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 Software
0x1bb  24 wb  32  . r  . .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 Software
0x1bc  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 Software
------------------------------------------------------------------------
  id  dep cl  bf lv rg d st  r  g  b  a F sr ax dp st  r  g  b  a ns  b cav
      visual  sz l  ci b ro  colorbuffer  gb bf th cl accumbuffer msaa  eat
------------------------------------------------------------------------
</pre>
</details>

### EGL configs

`EGL` was designed to make OpenGL context setup cross-platform (as much as possible),
and provides a similar API with function `eglChooseConfig()`, `eglGetConfigs()` and `eglGetConfigAttrib()`:

```cpp
  EGLDisplay aEglDisp = eglGetDisplay(EGL_DEFAULT_DISPLAY);
  if (aEglDisp == EGL_NO_DISPLAY) return;

  EGLint aNbConfigs = 0;
  eglGetConfigs(aEglDisp, nullptr, 0, &aNbConfigs);
  std::vector<EGLConfig> aConfigs(aNbConfigs);
  if (eglGetConfigs(aEglDisp, aConfigs.data(), aNbConfigs, &aNbConfigs) != EGL_TRUE)
    return;

  for (int aCfgIter = 0; aCfgIter < aNbConfigs; ++aCfgIter)
  {
    const EGLConfig aCfg = aConfigs[aCfgIter];
    EGLint aConfigId = 0, aCaveat = 0, aBufferSize = 0;
    eglGetConfigAttrib(aEglDisp, aCfg, EGL_CONFIG_ID, &aConfigId);
    eglGetConfigAttrib(aEglDisp, aCfg, EGL_CONFIG_CAVEAT, &aCaveat);
    eglGetConfigAttrib(aEglDisp, aCfg, EGL_BUFFER_SIZE, &aBufferSize);
    ...
  }
```

One peculiar innovation of `EGL` is an introduction of *Caveat* attribute with *None*, *Slow* or *NonConformant* values.
The usage of this attribute is, however, rather obscure.

*NonConformant* is supposed to mean that OpenGL (or another API supported by `EGL` like *OpenGL ES*) implementation
doesn't pass validation tests, but the only thing that application could do in this case is to avoid using this OpenGL implementation at all.

The *Slow* bit usually indicates that a particular pixel format is supported by OpenGL driver or hardware,
but has (considerable) performance implication, so that one should rather use another EGL config instead.
At the same time, apparently this bit is NOT set for software implementations of OpenGL, which naturally one would consider 'slow'.

Here is a sample listing of `EGL` implementation by *ANGLE* project on *Windows* platform used by browsers to implement *WebGL*:

<details><summary>EGL visuals</summary>
<pre>
[EGL] 240 EGL Configs
      visual  bf lv rg d st  colorbuffer  sr ax dp st accumbuffer msaa  cav
  id  dep cl  sz l  ci b ro  r  g  b  a F gb bf th cl  r  g  b  a ns  b eat
------------------------------------------------------------------------
0x097  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 None
0x0b5  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 None
0x0d3  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  0  .  .  .  .  0  0 None
0x0af  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  8  .  .  .  .  0  0 None
0x0cd  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  8  .  .  .  .  0  0 None
0x0eb  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  8  .  .  .  .  0  0 None
0x0a9  12 wb  16  . r  y .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 None
0x0c7  15 wb  16  . r  y .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 None
0x0e5  16 wb  16  . r  y .   5  6  5  0 .  .  . 16  0  .  .  .  .  0  0 None
0x0a3  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 None
0x0c1  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 None
0x0df  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  0  .  .  .  .  0  0 None
0x09d  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 None
0x0bb  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  8  .  .  .  .  0  0 None
0x0d9  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  8  .  .  .  .  0  0 None
0x098  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 None
0x0b6  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 None
0x0d4  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  0  .  .  .  .  0  0 None
0x0b0  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  8  .  .  .  .  0  0 None
0x0ce  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  8  .  .  .  .  0  0 None
0x0ec  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  8  .  .  .  .  0  0 None
0x0aa  12 wb  16  . r  y .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 None
0x0c8  15 wb  16  . r  y .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 None
0x0e6  16 wb  16  . r  y .   5  6  5  0 .  .  . 16  0  .  .  .  .  0  0 None
0x0a4  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 None
0x0c2  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 None
0x0e0  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  0  .  .  .  .  0  0 None
0x09e  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 None
0x0bc  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  8  .  .  .  .  0  0 None
0x0da  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  8  .  .  .  .  0  0 None
0x099  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 None
0x0b7  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 None
0x0b7  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 None
0x0d5  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  0  .  .  .  .  0  0 None
0x0b1  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  8  .  .  .  .  0  0 None
0x0cf  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  8  .  .  .  .  0  0 None
0x0ed  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  8  .  .  .  .  0  0 None
0x0ab  12 wb  16  . r  y .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 None
0x0c9  15 wb  16  . r  y .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 None
0x0e7  16 wb  16  . r  y .   5  6  5  0 .  .  . 16  0  .  .  .  .  0  0 None
0x0a5  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 None
0x0c3  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 None
0x0e1  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  0  .  .  .  .  0  0 None
0x09f  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 None
0x0bd  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  8  .  .  .  .  0  0 None
0x0db  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  8  .  .  .  .  0  0 None
0x09a  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 None
0x0b8  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 None
0x0d6  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  0  .  .  .  .  0  0 None
0x0b2  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  8  .  .  .  .  0  0 None
0x0d0  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  8  .  .  .  .  0  0 None
0x0ee  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  8  .  .  .  .  0  0 None
0x0ac  12 wb  16  . r  y .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 None
0x0ca  15 wb  16  . r  y .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 None
0x0e8  16 wb  16  . r  y .   5  6  5  0 .  .  . 16  0  .  .  .  .  0  0 None
0x0a6  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 None
0x0c4  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 None
0x0e2  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  0  .  .  .  .  0  0 None
0x0a0  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 None
0x0be  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  8  .  .  .  .  0  0 None
0x0dc  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  8  .  .  .  .  0  0 None
0x09b  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 None
0x0b9  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 None
0x0d7  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  0  .  .  .  .  0  0 None
0x0b3  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  8  .  .  .  .  0  0 None
0x0d1  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  8  .  .  .  .  0  0 None
0x0ef  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  8  .  .  .  .  0  0 None
0x0ad  12 wb  16  . r  y .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 None
0x0cb  15 wb  16  . r  y .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 None
0x0e9  16 wb  16  . r  y .   5  6  5  0 .  .  . 16  0  .  .  .  .  0  0 None
0x0a7  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 None
0x0c5  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 None
0x0e3  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  0  .  .  .  .  0  0 None
0x0a1  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 None
0x0bf  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  8  .  .  .  .  0  0 None
0x0dd  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  8  .  .  .  .  0  0 None
0x09c  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  0  .  .  .  .  0  0 None
0x0ba  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  0  .  .  .  .  0  0 None
0x0d8  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  0  .  .  .  .  0  0 None
0x0b4  12 wb  16  . r  y .   4  4  4  4 .  .  .  0  8  .  .  .  .  0  0 None
0x0d2  15 wb  16  . r  y .   5  5  5  1 .  .  .  0  8  .  .  .  .  0  0 None
0x0f0  16 wb  16  . r  y .   5  6  5  0 .  .  .  0  8  .  .  .  .  0  0 None
0x0ae  12 wb  16  . r  y .   4  4  4  4 .  .  . 16  0  .  .  .  .  0  0 None
0x0cc  15 wb  16  . r  y .   5  5  5  1 .  .  . 16  0  .  .  .  .  0  0 None
0x0ea  16 wb  16  . r  y .   5  6  5  0 .  .  . 16  0  .  .  .  .  0  0 None
0x0a8  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  0  .  .  .  .  0  0 None
0x0c6  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  0  .  .  .  .  0  0 None
0x0e4  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  0  .  .  .  .  0  0 None
0x0a2  12 wb  16  . r  y .   4  4  4  4 .  .  . 24  8  .  .  .  .  0  0 None
0x0c0  15 wb  16  . r  y .   5  5  5  1 .  .  . 24  8  .  .  .  .  0  0 None
0x0de  16 wb  16  . r  y .   5  6  5  0 .  .  . 24  8  .  .  .  .  0  0 None
0x03d  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  0  .  .  .  .  0  0 None
0x055  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  8  .  .  .  .  0  0 None
0x04f  24 wb  24  . r  y .   8  8  8  0 .  .  . 16  0  .  .  .  .  0  0 None
0x049  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 None
0x043  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 None
0x03e  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  0  .  .  .  .  0  0 None
0x056  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  8  .  .  .  .  0  0 None
0x050  24 wb  24  . r  y .   8  8  8  0 .  .  . 16  0  .  .  .  .  0  0 None
0x04a  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 None
0x044  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 None
0x03f  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  0  .  .  .  .  0  0 None
0x057  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  8  .  .  .  .  0  0 None
0x051  24 wb  24  . r  y .   8  8  8  0 .  .  . 16  0  .  .  .  .  0  0 None
0x04b  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 None
0x045  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 None
0x040  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  0  .  .  .  .  0  0 None
0x058  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  8  .  .  .  .  0  0 None
0x052  24 wb  24  . r  y .   8  8  8  0 .  .  . 16  0  .  .  .  .  0  0 None
0x04c  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 None
0x046  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 None
0x041  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  0  .  .  .  .  0  0 None
0x059  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  8  .  .  .  .  0  0 None
0x053  24 wb  24  . r  y .   8  8  8  0 .  .  . 16  0  .  .  .  .  0  0 None
0x04d  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 None
0x047  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 None
0x042  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  0  .  .  .  .  0  0 None
0x05a  24 wb  24  . r  y .   8  8  8  0 .  .  .  0  8  .  .  .  .  0  0 None
0x054  24 wb  24  . r  y .   8  8  8  0 .  .  . 16  0  .  .  .  .  0  0 None
0x04e  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  0  .  .  .  .  0  0 None
0x048  24 wb  24  . r  y .   8  8  8  0 .  .  . 24  8  .  .  .  .  0  0 None
0x001  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x01f  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x079  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
0x019  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x037  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x091  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  8  .  .  .  .  0  0 None
0x013  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x031  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x08b  30 wb  32  . r  y .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
0x00d  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x02b  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x085  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
0x007  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x025  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x07f  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
0x002  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x020  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x07a  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
0x01a  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x038  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x092  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  8  .  .  .  .  0  0 None
0x014  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x032  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x08c  30 wb  32  . r  y .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
0x00e  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x02c  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x086  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
0x008  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x026  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x080  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
0x003  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x021  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x07b  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
0x01b  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x039  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x093  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  8  .  .  .  .  0  0 None
0x015  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x033  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x08d  30 wb  32  . r  y .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
0x00f  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x02d  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x087  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
0x009  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x027  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x081  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
0x004  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x022  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x07c  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
0x01c  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x03a  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x094  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  8  .  .  .  .  0  0 None
0x016  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x034  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x08e  30 wb  32  . r  y .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
0x010  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x02e  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x088  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
0x00a  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x028  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x082  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
0x005  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x023  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x07d  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
0x01d  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x03b  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x095  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  8  .  .  .  .  0  0 None
0x017  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x035  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x08f  30 wb  32  . r  y .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
0x011  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x00f  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x02d  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x087  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
0x009  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x027  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x081  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
0x004  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x022  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x07c  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
0x01c  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x03a  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x094  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  8  .  .  .  .  0  0 None
0x016  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x034  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x08e  30 wb  32  . r  y .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
0x010  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x02e  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x088  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  0  .  .  .  .  0  0 None
0x00a  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x028  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  8  .  .  .  .  0  0 None
0x082  30 wb  32  . r  y .  10 10 10  2 .  .  . 24  8  .  .  .  .  0  0 None
0x005  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x023  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  0  .  .  .  .  0  0 None
0x07d  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  0  .  .  .  .  0  0 None
0x01d  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x03b  24 wb  32  . r  y .   8  8  8  8 .  .  .  0  8  .  .  .  .  0  0 None
0x095  30 wb  32  . r  y .  10 10 10  2 .  .  .  0  8  .  .  .  .  0  0 None
0x017  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x035  24 wb  32  . r  y .   8  8  8  8 .  .  . 16  0  .  .  .  .  0  0 None
0x08f  30 wb  32  . r  y .  10 10 10  2 .  .  . 16  0  .  .  .  .  0  0 None
0x011  24 wb  32  . r  y .   8  8  8  8 .  .  . 24  0  .  .  .  .  0  0 None
0x069  48 wb  64  . r  y .  16 16 16 16 y  .  . 24  0  .  .  .  .  0  0 None
0x063  48 wb  64  . r  y .  16 16 16 16 y  .  . 24  8  .  .  .  .  0  0 None
0x05e  48 wb  64  . r  y .  16 16 16 16 y  .  .  0  0  .  .  .  .  0  0 None
0x076  48 wb  64  . r  y .  16 16 16 16 y  .  .  0  8  .  .  .  .  0  0 None
0x070  48 wb  64  . r  y .  16 16 16 16 y  .  . 16  0  .  .  .  .  0  0 None
0x06a  48 wb  64  . r  y .  16 16 16 16 y  .  . 24  0  .  .  .  .  0  0 None
0x064  48 wb  64  . r  y .  16 16 16 16 y  .  . 24  8  .  .  .  .  0  0 None
0x05f  48 wb  64  . r  y .  16 16 16 16 y  .  .  0  0  .  .  .  .  0  0 None
0x077  48 wb  64  . r  y .  16 16 16 16 y  .  .  0  8  .  .  .  .  0  0 None
0x071  48 wb  64  . r  y .  16 16 16 16 y  .  . 16  0  .  .  .  .  0  0 None
0x06b  48 wb  64  . r  y .  16 16 16 16 y  .  . 24  0  .  .  .  .  0  0 None
0x065  48 wb  64  . r  y .  16 16 16 16 y  .  . 24  8  .  .  .  .  0  0 None
0x060  48 wb  64  . r  y .  16 16 16 16 y  .  .  0  0  .  .  .  .  0  0 None
0x078  48 wb  64  . r  y .  16 16 16 16 y  .  .  0  8  .  .  .  .  0  0 None
0x072  48 wb  64  . r  y .  16 16 16 16 y  .  . 16  0  .  .  .  .  0  0 None
0x06c  48 wb  64  . r  y .  16 16 16 16 y  .  . 24  0  .  .  .  .  0  0 None
0x066  48 wb  64  . r  y .  16 16 16 16 y  .  . 24  8  .  .  .  .  0  0 None
------------------------------------------------------------------------
  id  dep cl  bf lv rg d st  r  g  b  a F sr ax dp st  r  g  b  a ns  b cav
      visual  sz l  ci b ro  colorbuffer  gb bf th cl accumbuffer msaa  eat
------------------------------------------------------------------------
</pre>
</details>

### CGL pixel formats

*macOS* defines the class `NSOpenGLContext` on top of a lower-level `CGL` (Core OpenGL) interface.
Surprisingly, there are `CGLChoosePixelFormat()` and `CGLDescribePixelFormat()` functions similar to other APIs,
but there is NO interface for querying all supported pixel formats!

To be able to provide a listing similar to other platforms,
I have implemented a brute-force loop over possible (known to me) combinations of attributes,
and filtered out unique ones from returned supported configs:

```cpp
  // accelerated or software implementation
  for (int anAccel = 1; anAccel >= 0; --anAccel)
  {
    // 128 for float
    // 164 for half-float
    // 32  for R8G8B8A8
    // 30  for R10G10B10A2 (will be returned only when requested with 2 bits for alpha!)
    // 24 & 16 are never returned -> 32 is returned instead
    //
    // software implementation supports only two color formats: R8G8B8A8 and R32G32B32A32 (float)
    for (int aColor : {128, 64, 32, 30, 24, 16})
    {
      // 32 for float
      // 16 for half-float (accelerated-only)
      // 8  for R8G8B8A8
      // 2  for R10G10B10A2 (accelerated-only)
      // 0  alpha is never returned -> 8 is returned instead
      for (int anAlpha : {2, 8, 0})
      {
        // 32 corresponds to float?
        // 24 & 16 depth are never returned
        // 0  to disable depth (but should be combined with 0 stencil!)
        for (int aDepth : {32, 24, 16, 0})
        {
          // 8 for normal stencil
          // 0 to disable stencil
          for (int aStencil : {8, 0})
          {
            int anAttribs[] =
            {
              kCGLPFAColorSize, aColor,
              kCGLPFADepthSize, aDepth,
              ...
              0
            };
            CGLPixelFormatObj aFormat = nullptr;
            GLint aNbPixs = 0;
            if (CGLChoosePixelFormat((CGLPixelFormatAttribute* )anAttribs, &aFormat, &aNbPixs) != kCGLNoError)
              return;

            ...
```

In contrast to other platforms, *macOS* provides its own OpenGL implementation as part of a system that cannot be extended by a graphic driver.
As such, features and versions of OpenGL depend on *macOS* version - which is up to *OpenGL 4.1 Core Profile* as of today.
It also provides built-in software-emulated OpenGL.

The overall OpenGL support has been deprecated since *macOS 10.14* in favor of a proprietary *Metal* interface,
has somewhat degraded in quality (the OpenGL rendering should be now done only within the GUI thread;
there are some reports about artifacts within software-emulated OpenGL), but hasn't been removed yet.

Below is the sample listing on *Apple M4* and *macOS 26.4*.
One may notice the absence of legacy packed formats in listing below (like 16-bit color),
there are no formats without alpha channel, the depth buffer has only 32-bit depth configuration,
and there are full-float color formats.

<details><summary>CGL visuals</summary>
<pre>
[CGL] 78 CGL Visuals
      visual  bf lv rg d st  colorbuffer  sr ax dp st accumbuffer msaa  cav
  id  dep cl  sz l  ci b ro  r  g  b  a F gb bf th cl  r  g  b  a ns  b eat
------------------------------------------------------------------------
    0  96  . 128  . r  y .  32 32 32 32 y  .  . 32  8  .  .  .  .  0  0 None
    1  96  . 128  . r  y .  32 32 32 32 y  .  . 32  8  .  .  .  .  0  0 Software
    2  96  . 128  . r  y .  32 32 32 32 y  .  . 32  8  .  .  .  .  0  0 None
    3  96  . 128  . r  y .  32 32 32 32 y  .  . 32  8  .  .  .  .  0  0 Software
    4  96 wn 128  . r  y .  32 32 32 32 y  .  . 32  8  .  .  .  .  0  0 None
    5  96 wb 128  . r  y .  32 32 32 32 y  .  . 32  8  .  .  .  .  0  0 Software
    6  96  . 128  . r  y .  32 32 32 32 y  .  . 32  0  .  .  .  .  0  0 None
    7  96  . 128  . r  y .  32 32 32 32 y  .  . 32  0  .  .  .  .  0  0 Software
    8  96  . 128  . r  y .  32 32 32 32 y  .  . 32  0  .  .  .  .  0  0 None
    9  96  . 128  . r  y .  32 32 32 32 y  .  . 32  0  .  .  .  .  0  0 Software
   10  96 wn 128  . r  y .  32 32 32 32 y  .  . 32  0  .  .  .  .  0  0 None
   11  96 wb 128  . r  y .  32 32 32 32 y  .  . 32  0  .  .  .  .  0  0 Software
   12  96  . 128  . r  y .  32 32 32 32 y  .  .  0  0  .  .  .  .  0  0 None
   13  96  . 128  . r  y .  32 32 32 32 y  .  .  0  0  .  .  .  .  0  0 Software
   14  96  . 128  . r  y .  32 32 32 32 y  .  .  0  0  .  .  .  .  0  0 None
   15  96  . 128  . r  y .  32 32 32 32 y  .  .  0  0  .  .  .  .  0  0 Software
   16  96 wn 128  . r  y .  32 32 32 32 y  .  .  0  0  .  .  .  .  0  0 None
   17  96 wb 128  . r  y .  32 32 32 32 y  .  .  0  0  .  .  .  .  0  0 Software
   18  48  .  64  . r  y .  16 16 16 16 y  .  . 32  8  .  .  .  .  0  0 None
   19  48  .  64  . r  y .  16 16 16 16 y  .  . 32  8  .  .  .  .  0  0 None
   20  48 wn  64  . r  y .  16 16 16 16 y  .  . 32  8  .  .  .  .  0  0 None
   21  48  .  64  . r  y .  16 16 16 16 y  .  . 32  0  .  .  .  .  0  0 None
   22  48  .  64  . r  y .  16 16 16 16 y  .  . 32  0  .  .  .  .  0  0 None
   23  48 wn  64  . r  y .  16 16 16 16 y  .  . 32  0  .  .  .  .  0  0 None
   24  48  .  64  . r  y .  16 16 16 16 y  .  .  0  0  .  .  .  .  0  0 None
   25  48  .  64  . r  y .  16 16 16 16 y  .  .  0  0  .  .  .  .  0  0 None
   26  48 wn  64  . r  y .  16 16 16 16 y  .  .  0  0  .  .  .  .  0  0 None
   27  30  .  32  . r  y .  10 10 10  2 .  s  . 32  8  .  .  .  .  0  0 None
   28  24  .  32  . r  y .   8  8  8  8 .  s  . 32  8  .  .  .  .  0  0 Software
   29  30  .  32  . r  y .  10 10 10  2 .  s  . 32  8  .  .  .  .  0  0 None
   30  24  .  32  . r  y .   8  8  8  8 .  s  . 32  8  .  .  .  .  0  0 Software
   31  30 wn  32  . r  y .  10 10 10  2 .  s  . 32  8  .  .  .  .  0  0 None
   32  24 wb  32  . r  y .   8  8  8  8 .  s  . 32  8  .  .  .  .  0  0 Software
   33  30  .  32  . r  y .  10 10 10  2 .  s  . 32  0  .  .  .  .  0  0 None
   34  24  .  32  . r  y .   8  8  8  8 .  s  . 32  0  .  .  .  .  0  0 Software
   35  30  .  32  . r  y .  10 10 10  2 .  s  . 32  0  .  .  .  .  0  0 None
   36  24  .  32  . r  y .   8  8  8  8 .  s  . 32  0  .  .  .  .  0  0 Software
   37  30 wn  32  . r  y .  10 10 10  2 .  s  . 32  0  .  .  .  .  0  0 None
   38  24 wb  32  . r  y .   8  8  8  8 .  s  . 32  0  .  .  .  .  0  0 Software
   39  30  .  32  . r  y .  10 10 10  2 .  s  .  0  0  .  .  .  .  0  0 None
   40  24  .  32  . r  y .   8  8  8  8 .  s  .  0  0  .  .  .  .  0  0 Software
   41  30  .  32  . r  y .  10 10 10  2 .  s  .  0  0  .  .  .  .  0  0 None
   42  24  .  32  . r  y .   8  8  8  8 .  s  .  0  0  .  .  .  .  0  0 Software
   43  30 wn  32  . r  y .  10 10 10  2 .  s  .  0  0  .  .  .  .  0  0 None
   44  24 wb  32  . r  y .   8  8  8  8 .  s  .  0  0  .  .  .  .  0  0 Software
   45  24  .  32  . r  y .   8  8  8  8 .  s  . 32  8  .  .  .  .  0  0 None
   46  24  .  32  . r  y .   8  8  8  8 .  s  . 32  8  .  .  .  .  0  0 None
   47  24 wn  32  . r  y .   8  8  8  8 .  s  . 32  8  .  .  .  .  0  0 None
   48  24  .  32  . r  y .   8  8  8  8 .  s  . 32  0  .  .  .  .  0  0 None
   49  24  .  32  . r  y .   8  8  8  8 .  s  . 32  0  .  .  .  .  0  0 None
   50  24 wn  32  . r  y .   8  8  8  8 .  s  . 32  0  .  .  .  .  0  0 None
   51  24  .  32  . r  y .   8  8  8  8 .  s  .  0  0  .  .  .  .  0  0 None
   52  24  .  32  . r  y .   8  8  8  8 .  s  .  0  0  .  .  .  .  0  0 None
   53  24 wn  32  . r  y .   8  8  8  8 .  s  .  0  0  .  .  .  .  0  0 None
   54  96  . 128  . r  y .  32 32 32 32 y  .  . 32  8  .  .  .  .  0  0 Software
   55  96  . 128  . r  y .  32 32 32 32 y  .  . 32  8  .  .  .  .  0  0 Software
   56  96 wb 128  . r  y .  32 32 32 32 y  .  . 32  8  .  .  .  .  0  0 Software
   57  96  . 128  . r  y .  32 32 32 32 y  .  . 32  0  .  .  .  .  0  0 Software
   58  96  . 128  . r  y .  32 32 32 32 y  .  . 32  0  .  .  .  .  0  0 Software
   59  96 wb 128  . r  y .  32 32 32 32 y  .  . 32  0  .  .  .  .  0  0 Software
   60  96  . 128  . r  y .  32 32 32 32 y  .  .  0  8  .  .  .  .  0  0 Software
   61  96  . 128  . r  y .  32 32 32 32 y  .  .  0  8  .  .  .  .  0  0 Software
   62  96 wb 128  . r  y .  32 32 32 32 y  .  .  0  8  .  .  .  .  0  0 Software
   63  96  . 128  . r  y .  32 32 32 32 y  .  .  0  0  .  .  .  .  0  0 Software
   64  96  . 128  . r  y .  32 32 32 32 y  .  .  0  0  .  .  .  .  0  0 Software
   65  96 wb 128  . r  y .  32 32 32 32 y  .  .  0  0  .  .  .  .  0  0 Software
   66  24  .  32  . r  y .   8  8  8  8 .  s  . 32  8  .  .  .  .  0  0 Software
   67  24  .  32  . r  y .   8  8  8  8 .  s  . 32  8  .  .  .  .  0  0 Software
   68  24 wb  32  . r  y .   8  8  8  8 .  s  . 32  8  .  .  .  .  0  0 Software
   69  24  .  32  . r  y .   8  8  8  8 .  s  . 32  0  .  .  .  .  0  0 Software
   70  24  .  32  . r  y .   8  8  8  8 .  s  . 32  0  .  .  .  .  0  0 Software
   71  24 wb  32  . r  y .   8  8  8  8 .  s  . 32  0  .  .  .  .  0  0 Software
   72  24  .  32  . r  y .   8  8  8  8 .  s  .  0  8  .  .  .  .  0  0 Software
   73  24  .  32  . r  y .   8  8  8  8 .  s  .  0  8  .  .  .  .  0  0 Software
   74  24 wb  32  . r  y .   8  8  8  8 .  s  .  0  8  .  .  .  .  0  0 Software
   75  24  .  32  . r  y .   8  8  8  8 .  s  .  0  0  .  .  .  .  0  0 Software
   76  24  .  32  . r  y .   8  8  8  8 .  s  .  0  0  .  .  .  .  0  0 Software
   77  24 wb  32  . r  y .   8  8  8  8 .  s  .  0  0  .  .  .  .  0  0 Software
------------------------------------------------------------------------
  id  dep cl  bf lv rg d st  r  g  b  a F sr ax dp st  r  g  b  a ns  b cav
      visual  sz l  ci b ro  colorbuffer  gb bf th cl accumbuffer msaa  eat
------------------------------------------------------------------------
</pre>
</details>

## Additional buffers

### Double buffer

Most applications should use double-buffered configs (e.g. `PFD_DOUBLEBUFFER` in `WGL`)
to prepare a back-buffer offscreen and swap it onscreen without visual artifacts.

The triple buffer might be configured on some systems, but is rarely a good choice as it provides more delay to user input.

### Quad buffer

The stereoscopic configurations (e.g. `PFD_STEREO` in `WGL`) allows users to choose a quad-buffered stereoscopic
window format with dedicated buffers per left and right eye.
Such visuals provided support for stereoscopic hardware of the past like shutter-glasses or interlaced displays,
but mostly abandoned nowadays by OpenGL vendors save small exceptions for supporting HDMI-connected 3D displays.

### Overlays/Underlays

Windowing systems provide interfaces for creation of Overlays/Underlays planes.
These are not needed for common OpenGL applications and could be ignored by most users.

### Additional/offscreen buffers

A long time ago, the buffers allocated for a window itself were the only thing you have.
Nowadays, OpenGL application should certainly use *Frame Buffer Objects* (FBO) to manage additional buffers.

Modern OpenGL applications should just ignore buffer formats with *auxiliary* / *accumulated* buffers.

Pbuffers (pixel-buffers) was another attempt for a configurable offscreen rendering before FBO became mainstream.

> *OCCT* before *6.5.0* used to create a `BITMAP` (`CreateDIBSection()`)
> with a non-accelerated OpenGL context to perform an offscreen image dump on *Windows* platform
> (formats with `PFD_DRAW_TO_BITMAP` flag are never accelerated).
> Earlier versions of *sView* used `wglCreatePbufferEXT()` to create an offscreen *Pbuffer* (pixel buffer),
> a transient API that provided hardware-accelerated offscreen rendering before *Frame Buffer Objects* finally landed to OpenGL.

The same applies to *multi-sampling* pixel-formats - it is better to create an FBO with MSAA
and resolve samples on blitting to a window back-buffer.

## Depth+Stencil buffer

Most systems support a packed `depth24_stencil8` format suitable for most graphical tasks.

In some scenarios (like rendering of a distant landscape), a 24-bit depth buffer might have insufficient precision.
The 32-bit depth might help here, but usually the rendered should apply also other tricks.

When main rendering is done into *Frame Buffer Objects* (FBO),
there is not much reason to allocate a depth buffer for a window as only a color buffer will be used.

## Color buffer

Lets skip archaic indexed color (palette) formats, and pass through the useful ones.

### 16-bit High color

It might look weird to see unevenly distributed pixels formats like `r5g6b5` with an extra bit for green color component,
but back in the days of a [High Color](https://en.wikipedia.org/wiki/High_color) formats
(16-bit RGB, 15 bits are rounded up to *16* for optimal memory alignment anyway),
it has been seen like a small win to weed out an extra depth at least for one component
(and notice that human eye sees *more shades of green* than any other color).

There is no good reason to use these formats nowadays due to insufficient color precision.

### 24-bit True color

Most users should use [True color](https://en.wikipedia.org/wiki/Color_depth#True_color_(24-bit)) 24-bit formats `r8g8b8`,
normally extended to 32-bit for memory alignment either with additional alpha channel `r8g8b8a8` or with unused bits for padding.

### Alpha channel

Some window visuals might have an alpha channel.

It should be noted that for commonly used blending functions within OpenGL doesn't use a value
in target color buffer (`GL_DST_ALPHA`) in the equation:

```cpp
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
```

That means that a common OpenGL application doesn't need an alpha channel in neither window buffer nor in offscreen FBO.

However, the alpha channel in window buffer *might be* used by a window composer for transparency effects
(e.g. for client-side window decorations or for drawing an overlay window).
This might be enabled/disabled explicitly via a dedicated platform-specific API, or enabled by default to a developer's frustration.

One may avoid unexpected side effects just by requesting visuals without an alpha component.
Some systems, however, may not provide visuals without an alpha channel.

The most robust approach is to fill alpha channel with `1.0` and disable further writes into it within normal rendering:

```cpp
// clear color buffer and fill in alpha channel as opaque
glColorMask(GL_TRUE, GL_TRUE, GL_TRUE, GL_TRUE);
glClearColor(0.0, 0.0, 0.0, 1.0);
// disable writes into alpha channel and perform main rendering
glColorMask(GL_TRUE, GL_TRUE, GL_TRUE, GL_FALSE);
```

### 30-bit Deep color

[Deep Color](https://en.wikipedia.org/wiki/Color_depth#Deep_color_(30-bit)) is a format with *30* or more bits per color (excluding alpha).
The most common format is `r10g10b10a2`, which brings 10-bit per color component or *1024 grades* compared to just *256 grades* in *True color*.

One may notice insufficiency of *256 grades* of *True color* as banding artifacts by drawing single-color or gray color gradients on the entire screen.
Though in case of normal photos and common graphics this could be hardly noticed.

The benefits of higher color precision in a window buffer, however, require also a display with higher precision.
The most common LCD panels are limited to 8-bit per component or use even 6-bit precision with smoothing algorithms.
Most systems will allow selection of 30-bit visuals only when a capable display is actually connected.

### HDR / wide gamut color

It is possible to find half-float (`r16g16b16a16`) and full-float (`r32g32b32a32`) window formats on some systems.
Such formats would allow rendering of a wide gamut or HDR content directly into window,
but displaying such content requires an extra platform-specific setup, modifications in rendering pipeline, and HDR content.

By default, the buffer content written by OpenGL will be still considered to be in *sRGB* colorspace.
In *macOS* one may specify the colorspace of a window buffer through `NSWindow::setColorSpace` method:

```cpp
// this is what common OpenGL renderer should done by default
[myWindow setColorSpace: [NSColorSpace sRGBColorSpace]];

// P3 Display color space
[myWindow setColorSpace: [NSColorSpace displayP3ColorSpace]];
```

When using `EGL`, one may rely on extension [`EGL_KHR_gl_colorspace`](https://registry.khronos.org/EGL/extensions/KHR/EGL_KHR_gl_colorspace.txt)
and its siblings `EGL_EXT_gl_colorspace_display_p3`, `EGL_EXT_gl_colorspace_scrgb`, `EGL_EXT_gl_colorspace_bt2020_linear`
on supported platforms / capable displays.

Changing colorspace to [DCI-P3](https://en.wikipedia.org/wiki/DCI-P3) or other extended colorspaces
without changes in rendering pipeline will result in distorted oversaturated/dark/overbright colors.

