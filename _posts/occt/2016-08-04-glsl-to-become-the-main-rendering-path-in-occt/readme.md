---
layout: post
date: 2016-08-04
title: GLSL to become the main rendering path in OCCT
categories: occt
tags: glsl opencascade opengl opensource
permalink: /occt/2016-08-04-glsl-to-become-the-main-rendering-path-in-occt/
thumbnail:
author: Kirill Gavrilov Tartynskih
---

In the past, a graphic card in a computer was designed to perform a limited set of operations.
At some point, all cards had to provide the hardware-accelerated functionality called **Transformation and Lighting** (T&L).
Application developers had to implement all features and visual effects that were not parts of the fixed set of graphic card functionality on the CPU.

Later on, graphic cards made a dramatic step in design - they became programmable, being able to perform custom (initially very small and simple) programs.
This change started a new era of *Graphics Processing Units* - **GPU**.
Thus nowadays any modern computer (and even mobile device) has CPU (general-purpose computations) and GPU (processor dedicated for graphics acceleration).

This transition has been reflected in graphic libraries, including *OpenGL*.
The new approach has been named **Programmable Pipeline** while conventional approach is named **Fixed-Function Pipeline** or **FFP** (Transformation & Lighting providing limited functionality).

As GPUs had become fast enough, applications started using Programmable Pipeline for implementing real-time effects that were not possible before.
Games moved to the new pipeline very fast, since it allowed programming a visually appealing image.
In *CAD* software, this trend was less visible for a very simple reason - conventional *Fixed-Function pipeline* covered most features used by this software.

<!--break-->

## Evolution of GLSL in OCCT

Open CASCADE Technology started migration to **Programmable Pipeline** (*GLSL* programs) in '2013 ([#0024192](https://tracker.dev.opencascade.org/view.php?id=24192))
and shader support was first introduced in the OCCT 6.7.0 release.
Several objectives were considered, including:

- **Performance**.<br>
  *Programmable Pipeline* should show performance comparable to *Fixed-Function Pipeline*.
- **Flexibility**.<br>
  OCCT should provide an interface to application developers to implement custom *GLSL* programs.
- **Feature completeness**.<br>
  Programmable Pipeline should transparently replace the majority of visualization features already available in *OCCT*.
- **New features**.<br>
  Introduce new features not available using *Fixed-Function Pipeline*.
- **Compatibility**.<br>
  *OCCT* should keep a *Fixed-Function Pipeline* for some time.

**Flexibility**. In fact, the first release supported only custom (application-provided) *GLSL* programs with sample shaders in *OCCT*.
Only in further releases built-in *GLSL* programs were introduced as straight-forward replacement for *Fixed-Function Pipeline*.

**Feature completeness** is the main challenge on this way, because *OCCT* implements a lot of features using *Fixed-Function Pipeline*.
Some of them are quite obscure and rarely used, but still it is important to reach parity of available functionality between *Programmable* and *Fixed-Function Pipelines* in *OCCT*.

Open CASCADE Technology **6.8.0** introduced built-in *GLSL* programs covering main rendering paths, compatible with **OpenGL ES 2.0+** used on mobile platforms.
As a result, *Programmable Pipeline* became a replacement for *Fixed-Function Pipeline* (unavailable in OpenGL ES 2.0+)
and this change was fully transparent for applications, i.e. it did not require any changes in application code.
At this point, *Programmable Pipeline* was used mostly for compatibility with *Android* and *iOS* devices.

**Compatibility**, or keeping *Fixed-Function Pipeline* in parallel with *Programmable Pipeline*, increases code complexity and implies some performance penalty.
However, *OCCT* is not a graphics game (which has a relatively short, limited development cycle, and then just abandoned) - it is a framework used by numerous applications;
removing *Fixed-Function Pipeline* without implementing functional equivalence in *Programmable Pipeline* would be absolutely unacceptable for the users.
Mixing the two approaches allows supporting new features via *GLSL* while keeping old features via compatible code paths.

**New features**. *GLSL* programs allowed implementing features and new algorithms, not possible (or not as flexible and efficient) in *Fixed-Function Pipeline*.
Examples of such features:

- **Phong shading** (per-pixel lighting improving visual quality on rough triangulation).
- **Stereoscopic output**, including support of row-interlaced displays and anaglyph glasses (since OCCT **6.9.1**).
- **Ray-Tracing** built-in into *OCCT*.
  Ray-tracing is a powerful rendering approach for achieving photo-realistic visual quality.
  It is worth mentioning that Ray-Tracing can be easily enabled in the standard OCCT viewer.
  Moreover, *OCCT* supports a mixture of conventional (rasterization-based) approach and ray-tracing in the same view,
  which is rarely available in other Ray-Tracing rendering engines (usually designed for rendering static image or video).
- More features to come in the future.

## Deprecation of Fixed-Function Pipeline in OCCT

Open CASCADE Technology **7.0.0** is a major release increasing functional coverage of Programmable Pipeline to the level acceptable for most applications.
This release actually **deprecates** usage of *Fixed-Function Pipeline* in *OCCT* (as specified in Release Notes), although it is still used by default.
Applications might disable *Fixed-Function pipeline* by setting flag `OpenGl_Caps::ffpEnable` of `OpenGl_GraphicDriver::ChangeOptions()` before creating the viewer.

There are several reasons why *Fixed-Function Pipeline* has been deprecated in *OCCT*:

- This functionality was deprecated in OpenGL since version 3.1 that introduced the notion of *Core* (without deprecated functionality) and *Compatible Profiles*.
- Many new features are not available within *Compatible Profile* on some systems - *OS X* supports OpenGL 3+ only with disabled *Fixed-Function Pipeline*;
  OpenGL ES 2.0+ on mobile devices does not support it either;
  *Mesa* OpenGL driver on *Linux* supports OpenGL 3.1+ only without deprecated functionality.
- *GPU* vendors tend to abandon further support of *Fixed-Function Pipeline* in *OpenGL* drivers.
  This means that known bugs in this functionality will be never fixed (and new bugs appear), performance will not be improved.
- Moreover, a mixture of *Fixed* and *Programmable Pipelines* reveals new driver bugs.

Therefore, *OCCT* evolution is aimed to remove *Fixed-Function* rendering code, and deprecation is a first step.

## Current state and further steps

*Fixed-Function Pipeline* has already been disabled by default in the current development branch (`master`), and the next *OCCT* release will include this change.
*OCCT* still keeps old functionality, however new visualization features will not be propagated to *Fixed-Function Pipeline* and new bugs related to *Fixed-Function Pipeline* will be processed with low priority.

At some point, deprecated functionality will be completely removed from *OCCT* code base to improve maintainability.
