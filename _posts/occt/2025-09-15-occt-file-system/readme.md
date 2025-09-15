---
layout: post
date: 2025-09-15
title: File streams in OCCT
categories: occt
tags: c++ opencascade opensource
permalink: /occt/2025-09-15-occt-file-system/
thumbnail:
author: Kirill Gavrilov Tartynskih
---

Handling file input/output in C++ framework API looks pretty straightforward from the first glance,
but becomes more complicated when the developer wants to define custom streams, deal with virtual/remote files systems.
An experienced C++ developer might have seen plenty of C/C++ libraries exposing various stream APIs,
and multiple OCCT classes take standard C++ streams `std::istream`/`std::ostream` on input.

OCCT 7.6 introduced the new class `OSD_FileSystem`, which could be easily missed by the majority of OCCT users.
However, this class is intended to solve numerous problems related to file system operations in OCCT, and might be useful to be aware of it.

<!--break-->

## UNICODE file paths

The most straightforward way to expose file operations in C++ framework is to pass a file path to it as a *string*.
It might be `const char*` or `std::string`, or `TCollection_AsciiString` in case of OCCT:
```cpp
  bool Image_AlienPixMap::Save (const TCollection_AsciiString& theFileName);
```

Then the library opens a file stream using standard functions like `fopen()` or `std::fstream` and does whatever it wants.
The system API would normally expect the path to be given in a system locale, which is *UTF-8* in case of the modern *Linux*/*Unix* systems.

That will not, however, work smoothly on the *Windows* platform, which defines a dedicated set of 'wide' functions for dealing with *UNICODE* strings
(in *UTF-16*, like [`_wfopen()`](https://learn.microsoft.com/ru-ru/cpp/c-runtime-library/reference/fopen-s-wfopen-s)),
while 'legacy' functions (like `fopen()`) handle string in active [Windows code page](https://en.wikipedia.org/wiki/Windows_code_page).

As result, many C/C++ libraries expose two sets of functions - taking `const char*` as a file path and `const wchar_t*` (implemented only for *Windows*).
Apparently, duplicated methods bloat the libraries' API and don't look nice at all.
*FreeImage* library is a typical example of this approach:
```cpp
  FIBITMAP* FreeImage_Load (FREE_IMAGE_FORMAT fif, const char*    filename, int flags);
  FIBITMAP* FreeImage_LoadU(FREE_IMAGE_FORMAT fif, const wchar_t* filename, int flags);
```

The alternative could be [UTF-8 Everywhere](https://utf8everywhere.org/), adopted by many libraries like *FFmpeg* and *OCCT*.
Following this approach, the library expects and returns all strings given as `const char*` as *UTF-8* by default.
The library implicitly performs conversion between *UNICODE* encodings and redirects file routines to use 'wide' API on *Windows* platform specifically.

OCCT declares `TCollection_AsciiString` as a *UTF-8* string, while `TCollection_ExtendedString` defines a *UTF-16* string:
```cpp
  PCDM_ReaderStatus TDocStd_Application::Open (const TCollection_ExtendedString& thePath,
                                               Handle(TDocStd_Document)& theDoc,
                                               const Message_ProgressRange& theRange = Message_ProgressRange())
```

OCCT defines a set of wrappers in `OSD_OpenFile.hxx` for opening C/C++ file streams from *UNICODE* file paths in a common way (by handling *Windows* platform specifically):
```cpp
//! Function opens the file.
//! @param theName name of file encoded in UTF-8
//! @param theMode opening mode
//! @return file handle of opened file
FILE* OSD_OpenFile (const char* theName, const char* theMode);

//! Function opens the file stream.
//! @param theStream stream to open (e.g. std::ifstream/ofstream/fstream)
//! @param theName name of file encoded in UTF-8
//! @param theMode opening mode
template <typename T>
void OSD_OpenStream (T& theStream, const char* theName, const std::ios_base::openmode theMode);
```

## Standard file streams

Some libraries may take standard file stream like `FILE*` or `std::fstream` on input.
Then the library works with the passed stream using standard functions like `fread()`/`fwrite()`/`fseek()`.
Here is an example of such API *libpng*:
```cpp
  void png_init_io(png_structp png_ptr, FILE* fp);
```

This approach provides limited extensibility - user will be unable to wrap a custom stream into `FILE*` or into `std::fstream`.
Moreover, passing `FILE*` is troublesome on Windows platform with *MSVCRT*, where `FILE` structure is defined differently within *Debug* and *Release* runtimes,
leading to access violations in case of a mismatch.

As alternative to `FILE*`, the library might also accept an integer file descriptor - the most low-level unbuffered interface to files/sockets,
accessed via functions like `read()`/`write()`/`lseek()` (or from which user may create `FILE*` via `_fdopen()`/`fdopen()`).
This is very unusual scenario might happen when trying to resolve `ContentResolver` in Android SDK to work with the stream in C/C++:
```
  android.content.ContentResolver::openFileDescriptor() -> android.os.ParcelFileDescriptor::getFd();
```

The C++ framework may define interfaces taking basic virtual STL interfaces `std::istream`/`std::ostream`,
which would be more flexible than taking a specific implementations (like `std::ifstream`).
OCCT API has many examples of such interface:
```cpp
  void BRepTools::Dump (const TopoDS_Shape& theShape, std::ostream& theStream);
```

## Custom file streams

Using STL stream interfaces might look like a natural choice, but it might not be the best choice.
C++ STL stream interface is quite cumbersome on its own and might be tricky to implement properly. Moreover, this interface cannot be used in C API.
Finally, C++ STL stream might lack some useful interfaces for less common stream operations.

For these reasons, many libraries prefer to define their own stream interface as a set of functions, normally including *read*/*write*/*seek* operations.
Here is an example of such API in *FreeImage* library:
```cpp
  typedef void* fi_handle;
  typedef unsigned (DLL_CALLCONV *FI_ReadProc)  (void *buffer, unsigned size, unsigned count, fi_handle handle);
  typedef unsigned (DLL_CALLCONV *FI_WriteProc) (void *buffer, unsigned size, unsigned count, fi_handle handle);
  typedef int  (DLL_CALLCONV *FI_SeekProc) (fi_handle handle, long offset, int origin);
  typedef long (DLL_CALLCONV *FI_TellProc) (fi_handle handle);

  FI_STRUCT(FreeImageIO) {
    FI_ReadProc  read_proc;     //! pointer to the function used to read data
    FI_WriteProc write_proc;    //! pointer to the function used to write data
    FI_SeekProc  seek_proc;     //! pointer to the function used to seek
    FI_TellProc  tell_proc;     //! pointer to the function used to aquire the current position
  };
```

OCCT, however, decided to rely on a standard C++ stream interface defined by STL.

## File protocols and OSD_FileSystem

Some libraries are not limited to standard C/C++ file APIs (like `fopen()`), but support other protocols like `http://`, `https://`, `ftp://` and others.
For instance, `avio_open()` in *FFmpeg* initializes a custom stream API `AVIOContext` from URL that could be a local or remote file like `https://file.mkv`:
```cpp
  int avio_open (AVIOContext **s, const char *url, int flags);
```

A similar API could be found in OCCT - the interface `OSD_FileSystem` defines two virtual functions for opening STL input/output streams from the given file path:

```cpp
  virtual std::shared_ptr<std::istream> OpenIStream (const TCollection_AsciiString& theUrl,
                                                     const std::ios_base::openmode theMode,
                                                     const int64_t theOffset = 0,
                                                     const std::shared_ptr<std::istream>& theOldStream = std::shared_ptr<std::istream>());
  virtual std::shared_ptr<std::ostream> OpenOStream (const TCollection_AsciiString& theUrl,
                                                     const std::ios_base::openmode theMode);
```

It also provides the default implementation returned by `OSD_FileSystem::DefaultFileSystem()` relying on `OSD_OpenStream` described before.
`OSD_FileSystem` is designed to be configurable and extensible - it allows registering custom protocols:

```cpp
  static void AddDefaultProtocol   (const Handle(OSD_FileSystem)& theFileSystem, bool theIsPreferred = false);
  static void RemoveDefaultProtocol(const Handle(OSD_FileSystem)& theFileSystem);
```

The user may now register its own protocols for accessing file streams from various remote resources
(like a web-server, local network file server, cloud file storage service),
from in-memory resources and from encrypted resources.

OCCT also provides `OSD_CachedFileSystem` helping algorithms to reuse a set of already opened file streams.
The cache is useful while reading file formats like [glTF](https://www.khronos.org/gltf/) (see `RWGltf_CafReader` implementation) or *JT*.
These formats define numerous references to data chunks in (external) binary file(s), so that a reader has to open the same file many times.
On many platforms, file opening operation might be quite expensive (like opening a model from *Samba* file server on *Windows*)
and drop performance of operation without a file stream cache (from a couple of seconds to minutes!).

`OSD_FileSystem` also provides an answer to an API problem for passing custom streams to multi-file formats like *STEP* or *glTF*.
Lets consider that we pass a custom input stream to a STEP file defining external references:
```cpp
  IFSelect_ReturnStatus STEPCAFControl_Reader::ReadStream (const Standard_CString theName,
                                                           std::istream& theIStream);
```
But then what? STEP reader will find external references in the main file and will try to open and read them using standard file operations...
Instead of this, one may now define a custom `OSD_FileSystem` implementation that will create custom file streams using whatever implementation the user wants!

## File systems and OSD_FileSystem flows

Reading and writing file streams is only subset of the operations to be done on files.
In many cases, applications need to navigate through folders, read file properties (like timestamps, a or file size), delete files, folders and do other things.

C++17 introduced `std::filesystem`, which made many standard file operations available in a cross-platform way.
But this wasn't available when OCCT development has been started, and user may find
`OSD_Path`, `OSD_File`, `OSD_FileIterator`, `OSD_Directory`, `OSD_DirectoryIterator` classes wrapping platform-specific APIs.

One may consider `std::filesystem` as a direct replacement to `OSD_FileSystem`, but apparently it is unsuitable for that.
`OSD_FileSystem` provides an interface for registering new protocols, while `std::filesystem` allows accessing only the local file system.

`OSD_FileSystem` is still in early state in OCCT and has things to be improved:
- Extend API with more file-system-alike methods, like listing folders, copying/moving files and others.
  The implementation might shift towards `std::filesystem` usage for local filesystem,
  and `OSD_FileIterator` and similar legacy classes should uniformly switch to `OSD_FileSystem` for consistency.
- Adopt local customizations of `OSD_FileSystem` within specific algorithms to avoid altering the global instance.
  This could be done by exposing `OSD_FileSystem` in an API of specific algorithms (like STEP translator).
- Improve documentation to make `OSD_FileSystem` design intent more clear to end-users.
