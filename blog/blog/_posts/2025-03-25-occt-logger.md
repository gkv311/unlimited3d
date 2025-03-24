---
layout: post
title: 2025-03-25 OCCT Communication language
categories: en blog
tags: draft
permalink: /blog/2025/03/25/occt-communication-language/
---

Users sometimes got surprised that OCCT framework provides logging mechanism - `Message_Messenger`.
When algorithm doesn't crash your application and doesn't raise unhandled C++ exceptions - it doesn't mean that algorithm worked as expected.

Logging is a powerful mechanism that helps developers and users to see what is going on in the framework,
to learn how algorithms work, to diagnose issues in your own code, and to report bugs to developers with a meaningful description.

This is one of the languages used by framework to communicate with it's developers and users, which is quite helpful to learn.

<!--break-->

Every complex system implements logging in one way or another - like `av_log()` in *FFmpeg*, `vtkLogger` in *VTK*,
`__android_log_write()` in *Android NDK*, `ReportEvent()` in *WinAPI*.
So does OCCT with its `Message::DefaultMessenger()` interface.

This is especially important for applications without GUI,
but even graphical applications like CAD Assistant could provide message log widget with more detailed and verbose information about events.

Starting from simple messages to `std::cout` and `std::cerr`,
developers soon realize that this is not flexible enough and invent various common tools for logging purposes.

```
  std::cout << "Trace: doSomething()\n";
  doSomething();
```

Usually developers introduce verbosity levels to filter out messages by their importance,
beautify output, include timestamps, message context, redirect logging into the file,
make some message to depend on compiler options (e.g. enable some verbose output only in debug builds), etc.

## OCCT Messenger

TODO

Logging facilities in OCCT is a compromise between simplicity and complexity.
To print message into the main messenger, developer should define its gravity from `Message_Gravity` enumeration and write something like this:

```.cpp
// debug message Message_Trace
Message::SendTrace() << "MyAlgo: diagnostics message for developers, most users will never see it.";
// info message Message_Info
Message::SendInfo() << "MyAlgo: algorithm just reports about success / completion / status.";
// warning message Message_Warning
Message::SendWarning() << "MyAlgo: algorithm is OK, but there are some warnings to the user.";
// important warning message Message_Alarm
Message::SendAlarm() << "MyAlgo: something not good happened.";
// error message Message_Fail
Message::SendFail() << "MyAlgo: failure happened.";
// the same as Message::SendWarning()
//Message::Send(Message_Warning) << "MyAlgo: warning message.";
```

If you'll run the code above in Draw Harness application, then you'll see something like this:

TODO  screenshot

Notice that messages are nicely colored, so that you may easily distinguish their importance, but one message is missing.
This is because verbose messages `Message_Trace` are filtered out by default, and if we will raise trace level `dtracelevel trace`, we'll see all messages:

TODO  screenshot

OCCT logging mechanism consists of two main parts - messenger `Message_Messenger` and printers `Message_Printer`.
Messenger `Message_Messenger` is an entry point where messages are sent, e.g. execution context.

Some algorithms provide interface to pass Messenger instance, but most of them just put messages into global Messenger returned by `Message::DefaultMessenger()`.
The main task of Messenger is to dispatch messages to Printers registered in `Message_Messenger::Printers()` list,
while `Message_Printer` defines actual message printing facilities via virtual `Message_Printer::send()` interface.

OCCT provides several standard `Message_Printer` implementations:
- `Message_PrinterOStream` prints messages to `std::cout` stream or into the file;
- `Message_PrinterSystemLog` prints messages to OS-specific system log (`syslog()` on UNIX/Linux, `__android_log_write()` on Android, `ReportEvent()` on Windows).

Each `Message_Printer` instance stores a property `Message_Printer::GetTraceLevel()` defining the lowest gravity of messages to be logged.

On startup, OCCT will initialize `Message::DefaultMessenger()` with only one single printer `Message_PrinterOStream`
redirecting messages of level `Message_Info` and higher to `std::cout` (this effectively filters out `Message_Trace` message by default).

But then you realize that your GUI application has no any terminal, how user would see the messages?


## XDE Messenger

TODO
