---
layout: post
date: 2019-09-15
title: GL_R16 and OpenGL ES on Android
categories: sview
tags: android opengl sview
permalink: /sview/2019-09-15-gl_r16-and-opengl-es-on-android/
thumbnail:
---

*sView* user on *Android* device just sent me a link to online stereoscopic 3D video with `yuv420p10` pixel format... and it is not an *anime*.
The user complained about green screen in the player instead of a video.

So far I haven't seen much videos with more than *8-bit per channel* in a wild.
Appearance of such files in online services surprised me, and I've started investigation of this problem.

<!--break-->

*FFmpeg* decodes such videos into pixel formats like `AV_PIX_FMT_YUV420P10LE`.
This pixel format has *10-bit per-channel* payload, but actually aligned into *16-bit per-channel* planes.
*sView* uploads such pixel formats into `GL_ALPHA16` textures, which has been implemented a long time ago.
After some investigation, I figured out that there is no *OpenGL ES* implementation supporting such texture format,
and the only direct counterpart is `GL_R16_EXT` (normalized *16-bit red channel only* format, same as `GL_R16` on desktop),
which is not yet in any *core OpenGL ES* specs, and requires extension [`GL_EXT_texture_norm16`](https://www.khronos.org/registry/OpenGL/extensions/EXT/EXT_texture_norm16.txt).

By the way, all this "deprecation thing" applied to useful texture formats like `GL_APHA8`, replaced by `GL_R8` for no profit within *OpenGL 3+* on desktop, always confused me.
Applications supporting wide range of *OpenGL* drivers (including *OpenGL 2.1*) has to become messed up with different code per *OpenGL* version for exactly the same thing!
As texture swizzling came later (and also messed-up API), fallback requires dummy *GLSL* code modifications fetching `.r` or `.a` of texture sample color.

Apparently, so far just two vendors support `GL_EXT_texture_norm16` extension – *Qualcomm Adreno* and *NVIDIA Tegra*,
which is quite predictable considering links to desktop graphics in their past.
Testing on *Adreno* device has shown good playback performance of given video sample.

Unfortunately, devices with *Mali* graphics do not support this extension, leading to bad performance due to software conversion via `SWScaler`.
Possible solution would be uploading planes into `GL_R16UI` textures (which is included into *OpenGL ES 3.0* specs,
but *integer formats* do not support texture filtering) and either implement texture filtering via *GLSL* program or convert it further into `GL_R32F` format
(the latter one seems to be implemented by some Internet browsers).
