---
layout: post
date: 2021-11-30
title: OCCT and legacy compilers
categories: occt
tags: c++ opencascade opensource poll statistics
permalink: /occt/2021-11-30-occt-and-legacy-compilers/
thumbnail:
author: Kirill Gavrilov Tartynskih
---

*Open CASCADE Technology* works on a vast range of platforms, and it is one of high priorities to keep supporting major compilers.

I have prepared a short anonymous [questionnaire](https://docs.google.com/forms/d/e/1FAIpQLSdkHDJA4RBaz1YewLtvztKmox4i9m4XMpVTUdaT760-xPyXzQ/viewform) to collect some statistics,
and you are welcome to share your opinion on this [forum thread](https://dev.opencascade.org/content/dropping-support-obsolete-compilers-occt-77).

<!--break-->

As you may notice, *OCCT 7.5.0* still could be built with *Visual Studio 2008*, and *OCCT 7.6.0* supports *Visual Studio 2010+*, while many projects rapidly drop support of "old" compilers.
For instance, [Qt 6.2 requires VS2019](https://doc-snapshots.qt.io/qt6-6.2/supported-platforms.html) for building.

New compilers bring new features, but practically speaking *C++11* is a good base for development,
and I could barely see what kind of a breaking new feature of *C++14*/*C++17*/*C++20* would make such mature project as *OCCT*
suddenly much better from end-user point of view or would improve it's development pace.
Still, supporting legacy compilers with *incomplete C++11* support (like *VS2010*-*VS2013*) is an extra maintenance burden to the project, and an awkward restriction to new developers.

For instance, *OCCT* defines auxiliary macros `Standard_OVERRIDE` instead of direct usage of `override` for virtual methods for compatibility with old compilers,
and there is still a boilerplate like `opencascade::std::shared_ptr` in the code for compatibility with ancient *VS2008*!
Many patches have been revised to be integrated into *OCCT* for the same reasons, wasting extra developer's time to figure out why modern C++ code doesn't compile.

Personally, *VS2015* looks like a most reliable minimal compiler version to work with nowadays, and I can barely see any use cases of projects still using *VS2010*-*VS2013*.
Well, I think that there are still many projects using these and even older versions of *Visual Studio*, but they are in maintenance state and are not upgraded to newer *OCCT* releases anyway.
