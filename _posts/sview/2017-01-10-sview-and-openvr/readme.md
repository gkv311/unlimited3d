---
layout: post
date: 2017-01-10
title: sView and OpenVR
categories: sview
tags: openal opensource stereoscopic sview vr
permalink: /sview/2017-01-10-sview-and-openvr/
thumbnail:
---

[sView 17.01](https://sview.ru/en/download/) has been released recently introducing *OpenVR* support and various improvements for viewing *VR360* panoramas.

*OpenVR* support allows viewing videos and images in *HMD* devices like *HTC Vive* – with head tracking support within *VR360* panoramas.

<!--break-->

New version also introduces support for audio tracking within panoramic videos – a very nice feature for watching video in *HMD*.
This feature requires multi-channel audio streams (it will work with stereo stream, but spatial effect will be very poor in this case).
*sView* now also supports a 4-channel *Ambisonic B-Format* audio streams – as it supported by [OpenAL Soft library](https://www.openal-soft.org/) through `AL_EXT_BFORMAT` extension.
*Ambisonic audio* is actually a requirement for uploading videos with [spatial audio onto YouTube](https://support.google.com/youtube/answer/6395969?hl=en).
Using external tools it is also possible to download *VR360* videos with spatial audio from *YouTube* and play it in *sView*.

User *Ulf Brusquini* has prepared a small tutorial for configuring spatial audio playback in *sView*:

[![video](https://img.youtube.com/vi/7W4HwvMU6MQ/0.jpg)](https://www.youtube.com/watch?v=7W4HwvMU6MQ)

One curious thing about [OpenVR](https://github.com/ValveSoftware/openvr) is that right now it is not very *"open"* as it can be deduced from name.
In fact it is only [SteamVR](https://store.steampowered.com/steamvr) which can be used with it (and thus requires *Steam* client to be installed),
and *OpenVR* itself is a draft for open API with only one implementation having no source code.

*OpenVR* is a *Valve* initiative for making *VR* open to everybody (*HTC Vive* has been developed by *HTC* with *Valve* in partnership).
And now future of open VR standard in hands of [Khronos group](https://www.khronos.org/news/press/khronos-announces-vr-standards-initiative) – well known consortium developing open standards (like *OpenGL*).
So developers might expect a really open standard which will be accepted by many VR vendors, instead of mess of vendor-locked APIs currently available.
