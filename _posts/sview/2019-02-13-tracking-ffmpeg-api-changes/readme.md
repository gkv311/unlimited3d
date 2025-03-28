---
layout: post
date: 2019-02-13
title: Tracking FFmpeg API changes
categories: sview
tags: c++ ffmpeg sview
permalink: /sview/2019-02-13-tracking-ffmpeg-api-changes/
thumbnail:
---

Using external libraries in a project as well as writing your own library reveal *API/ABI* compatibility issues.
It is difficult preserving a public *API* of an intensively developed library, and almost impossible managing *ABI* compatibility.

In this article I'm just sharing my thoughts about maintenance burden to keep *sView* source code compatible with various *Linux* (*Ubuntu*) releases depending on *FFmpeg* version.

<!--break-->

**API**, ***Application Programming Interface compatibility*** of two libraries means that you can replace one library with another,
***rebuild your application*** without noticing difference (without need to modify your source code).
Usually, this is done in one direction – moving to the newer library version doesn't cause issues, while trying to jump onto older one might cause problems.
This is because the most usual way to keep *API* compatibility is adding a new one while still preserving old.
Let see how it looks with *FFmpeg*:

```cpp
    attribute_deprecated
    int avcodec_decode_audio3(AVCodecContext *avctx,
                              int16_t *samples,
                              int *frame_size_ptr,
                              AVPacket *avpkt);

    int avcodec_decode_audio4(AVCodecContext *avctx, AVFrame *frame,
                              int *got_frame_ptr, const AVPacket *avpkt);
```

So we have two functions with one marked ***deprecated***.
You may notice funny numbers at function name – it is correct, *FFmpeg* had *4* versions of `avcodec_decode_audioN()` function
and all of them are now deprecated or removed – replaced by even newer *API* `avcodec_send_packet()`/`avcodec_receive_frame()`.

Deprecation is very helpful mechanism in programming – it gives some time to library users to postpone porting headache and still use newest version of the library (sometimes with some limitations).
Even more, well-defined deprecation warning gives developer a good porting hint instead of not helpful compiler error *"method doesn't exist"*,
and allows porting step by step and see working application in progress rather than all in one (fix all compiler errors and crashes).

```cpp
    #if defined(_MSC_VER)
      #define attribute_deprecated __declspec(deprecated)
    #else // gcc, clang
      #define attribute_deprecated __attribute__((deprecated))
    #endif
```

**ABI**, ***Application Binary Interface compatibility*** of two libraries means that you can replace one with another without rebuilding application at all.
This is also usually managed only one direction (you can run application on newer version of the library), and it is even more difficult to preserve in library development.
For most libraries, this is done by publishing a corrective releases with critical bug-fixes, while new releases are packaged with version suffix (hence, *ABI* incompatible).
Only rare libraries and projects are hunting *ABI* compatibility, like [glibc](https://abi-laboratory.pro/?view=timeline&l=glibc).

Breaking ABI is very easy, even without removing any functions. Consider you need adding a new item `PixelFormat_RGBA8` into existing enumeration:

```cpp
    enum PixelFormat {
      PixelFormat_None,
      PixelFormat_RGB8,
      PixelFormat_RGBA8,    //!< new item
      PixelFormat_YUV420P,
      PixelFormat_YUV444P,
    };
```

`PixelFormat_RGBA8` should follow existing item `PixelFormat_RGB8`, but placing it there will lead to *ABI* breakage,
because values of old enumeration items following `PixelFormat_RGB8` will be changed.
In case of C++ classes, there are much more hidden *ABI* breaking issues.

***Application source code compatibility*** with different library releases allows distributing the same application across wide range of platforms.
In many cases, application developer builds all dependencies (external libraries) and include them within distribution package,
so that application code can be bound to specific versions of these libraries.
However, this approach doesn't fit well into Linux-like systems with package manager,
where each library (or framework) is expected to be put into dedicated package, so that it can be shared across different applications.

One solution to this problem is distributing application with platform-specific patches, solving compatibility issues with system-provided versions of libraries.
It can be done for different platform releases, or even for different distributions.
For instance, *Debian* source packages (`.deb`) allows putting a set of vendor-specific patches, which can be applied depending on distribution platform.
This allows making a single source package with platform-specific patches applied automatically by binary package building system,
but leads to extra maintenance issues (patches are NOT part of original project) and mess up of actual source code across various platforms.
This was a common practice for putting *Ubuntu*-specific patches into *Debian* packages,
which has been considered destructive, so that [Debian technical committee decided to forbid such packages](https://lists.debian.org/debian-devel-announce/2018/11/msg00004.html) in future.

An alternative approach is putting platform-specific (or version-specific) modifications directly into application source code.
*FFmpeg* is intensively developed framework with regular *API* changes in history,
so that maintaining *sView* compatibility across various *Linux* distributions is a heck of a problem.
It is still possible locking project onto specific *FFmpeg* version, but this would mean that releases on older *Linux* platforms will not receive *sView* updates,
or would require extra user efforts (installing newer *FFmpeg* libraries from other repositories), which doesn’t sounds fare.
Even if we ignore a complete list of *Ubuntu* releases (which comes to life every *6 months*)'
there are also so-called **LTS** (*Long-term support*) releases coming with old system libraries,
but still popular across users – there are a lot of people NOT enjoying system mess up every year.

It some point, it was decided that *sView* source code will keep compatibility with old *FFmpeg* releases as long as possible,
and [Launchpad repository](https://launchpad.net/~sview/+archive/ubuntu/stable) is used to decide which releases to support.
*Launchpad* keeps already uploaded binary packages endlessly, but only limited list of *Ubuntu* releases can receive new/updated packages on *Launchpad*.
I've been unable finding this list on *Launchpad* itself (obviously, it somehow correlates with official *Ubuntu* releases support timeline),
so that I just try to upload everything and check what was accepted.
At the moment of righting this article, this list starts with *Ubuntu 16.04 LTS*.

The result of this hunt during last decade can be seen directly in *sView* source code.
Just take a look onto this code quotation decoding an audio package:

```cpp
            #if(LIBAVCODEC_VERSION_INT >= AV_VERSION_INT(57, 106, 102))
                if(toSendPacket) {
                    const int aRes = avcodec_send_packet(myCodecCtx, thePacket->getAVpkt());
                    if(aRes < 0 && aRes != AVERROR_EOF) {
                        anAudioPktSize = 0;
                        break;
                    }
                    toSendPacket = false;
                }

                myFrame.reset();
                const int aRes2 = avcodec_receive_frame(myCodecCtx, myFrame.Frame);
                if(aRes2 < 0) {
                    anAudioPktSize = 0;
                    break;
                }
                isGotFrame = 1;
                int aLen1 = 0; // dummy for code compatible with old syntax
            #elif(LIBAVCODEC_VERSION_INT >= AV_VERSION_INT(52, 23, 0))
                StAVPacket anAvPkt;
                anAvPkt.getAVpkt()->data = (uint8_t* )anAudioPktData;
                anAvPkt.getAVpkt()->size = anAudioPktSize;
                (void )toSendPacket;

                #if(LIBAVCODEC_VERSION_INT >= AV_VERSION_INT(53, 40, 0))
                (void )aDataSize;
                myFrame.reset();
                const int aLen1 = avcodec_decode_audio4(myCodecCtx, myFrame.Frame,
                                                        &isGotFrame, anAvPkt.getAVpkt());
                #else
                const int aLen1 = avcodec_decode_audio3(myCodecCtx,
                                                        (int16_t* )myBufferSrc.getPlane(0), &aDataSize,
                                                        anAvPkt.getAVpkt());
                #endif
            #else
                const int aLen1 = avcodec_decode_audio2(myCodecCtx,
                                                        (int16_t* )myBufferSrc.getPlane(0), &aDataSize,
                                                        anAudioPktData, anAudioPktSize);
            #endif
```

Such a code mess! This is just a top of an iceberg, and the whole *sView* source code (of course, only the part actually dealing with *FFmpeg*)
become a maze for handling different *FFmpeg* versions – enumerations renames,
functions renames, new enumeration values (image pixel formats, audio sample formats, codecs),
*LibAV* fork (which is impossible to detect in source code without a hack),
a list of decoding functions, metadata and stereoscopic data retrieval, thread-safety evolution, planar audio and many more...

Does it worth making a source code of your own project a garbage of `#ifdef`'s? I'm not sure of it...
It is even difficult to clean up code basing on specific *FFmpeg* release,
because for historical reasons this project defines dedicated versions per-library rather than for entire framework.
