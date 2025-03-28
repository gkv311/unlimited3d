---
layout: post
date: 2018-10-28
title: wglinfo – glxinfo analog for Windows
categories: tools
tags: opengl opensource tools
permalink: /tools/2018-10-28-wglinfo-glxinfo-analog-for-windows/
thumbnail:
---

Here is a my version of [wglinfo](https://github.com/gkv311/wglinfo/releases) - a command-line tool
for dumping information about *OpenGL* context on *Windows*, similar to `glxinfo` doing the job on *Linux*.

<!--break-->

Every developer of 3D applications have to know which graphics driver is currently installed on system.
And not just developer, this information might be pretty helpful even for normal user for troubleshooting issues with 3D applications.
Knowing that graphics driver is installed in system sometimes doesn't indicate that they are fully functional – in particular,
on *Windows* platform *Direct3D* application might work well, while *OpenGL* applications don't.
While there is a plenty diagnostic tools available for that purpose, command-line tools might be more handful in many cases.

`glxinfo` is a pretty small but helpful command-line tool on *Linux*, well-known by both professional and common users.
It's purpose is very simple – to print diagnostic information about *OpenGL* graphic driver installed on the system.
*OpenGL* version and graphics driver vendor are most useful from the output, although for developer every printed line could be helpful (like the list of available *OpenGL extensions*).

Installing `glxinfo` is quite straightforward on *Linux* (just `"sudo apt-get install glxinfo"` on any *Debian*-based distributive), but *Windows* lacks such a helpful tool at hand.
Curious users, however, should be aware of `wglinfo` – an analog of `glxinfo` for *Windows* which can be downloaded from some sites.
Unfortunately, the output of original tool lacks some important details like a list of `WGL` extensions and lacks updates for a very long time.

After struggling once again with this problem, I've decided developing and publishing a slightly "upgraded" version of `wglinfo` on *GitHub*, which now outputs more details about *OpenGL*:
https://github.com/gkv311/wglinfo/releases

In addition to *OpenGL* provided by system (`WGL`), it also prints information using `EGL` (`libEGL.dll`), if it is available.

Here is an example of output of this tool (with long lists truncated):

```
[WGL] WGL extensions:
WGL_ARB_extensions_string, WGL_ARB_pixel_format...

[WGL] OpenGL vendor string: ATI Technologies Inc.
[WGL] OpenGL renderer string: AMD Radeon (TM) R9 380 Series
[WGL] OpenGL version string: 4.5.13507 Compatibility Profile Context 23.20.15033.5003
[WGL] OpenGL shading language version string: 4.50
[WGL] OpenGL GPU memory: 2048 MiB
[WGL] OpenGL extensions:
GL_AMDX_debug_output, GL_AMD_blend_minmax_factor...

[WGL] OpenGL (core profile) vendor string: ATI Technologies Inc.
[WGL] OpenGL (core profile) renderer string: AMD Radeon (TM) R9 380 Series
[WGL] OpenGL (core profile) version string: 4.5.13507 Core Profile Context 23.20.15033.5003
[WGL] OpenGL (core profile) shading language version string: 4.50
[WGL] OpenGL (core profile) extensions:
GL_AMDX_debug_output, GL_AMD_blend_minmax_factor...

[WGL] OpenGL (software) vendor string: Microsoft Corporation
[WGL] OpenGL (software) renderer string: GDI Generic
[WGL] OpenGL (software) version string: 1.1.0
[WGL] OpenGL (software) extensions:
GL_WIN_swap_hint, GL_EXT_bgra, GL_EXT_paletted_texture.


[WGL] 396 WGL Visuals
visual x bf lv rg d st r g b a ax dp st accum buffs ms
id dep cl sp sz l ci b ro sz sz sz sz bf th cl r g b a ns b
------------------------------------------------------------------
0x001 32 wn . 32 . r . . 8 8 8 8 4 . . . . . . . .
0x002 32 wn . 32 . r . y 8 8 8 8 4 . . . . . . . .
..
------------------------------------------------------------------
visual x bf lv rg d st r g b a ax dp st accum buffs ms
id dep cl sp sz l ci b ro sz sz sz sz bf th cl r g b a ns b
------------------------------------------------------------------

[EGL] EGLVersion: 1.4 (ANGLE 2.1.0.46ad513f4e5b)
[EGL] EGLVendor: Google Inc. (adapter LUID: 000000000000a804)
[EGL] EGLClientAPIs: OpenGL_ES
[EGL] EGL extensions:
EGL_EXT_create_context_robustness, EGL_ANGLE_d3d_share_handle_client_buffer...

[EGL] OpenGL ES vendor string: Google Inc.
[EGL] OpenGL ES renderer string: ANGLE (AMD Radeon (TM) R9 380 Series Direct3D11 vs_5_0 ps_5_0)
[EGL] OpenGL ES version string: OpenGL ES 2.0 (ANGLE 2.1.0.46ad513f4e5b)
[EGL] OpenGL ES shading language version string: OpenGL ES GLSL ES 1.00 (ANGLE 2.1.0.46ad513f4e5b)
[EGL] OpenGL ES extensions:
GL_OES_element_index_uint, GL_OES_packed_depth_stencil...
```
