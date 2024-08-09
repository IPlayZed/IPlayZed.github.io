+++
title = "What is the difference between .NET and .NET Framework? AKA Microsoft has a bad naming sense."
date = 2024-08-05
updated = 2024-08-05
description = ""
[taxonomies]
tags = ["C#","microsoft",".NET"]
+++

## Prologue

I do like working in C#. It is an elegant language, memory safe, it will run on any platform where the Common Language Runtime is available. It is so backwards compatible, that with some magic, you [could port any Windows app to Windows 95](https://youtu.be/CTUMNtKQLl8?si=XgFkojr5OULyg9Yx). Matt is crazy good that he managed to do this and the video is cinema production quality... so go and watch it.

Anyhow, dotnet versions exist, and that is a fact. Different version support different things. A .NET standard defines the APIs the framework provides. For each .NET version there are a set of languages from which their respective compiler can create a so called CIL, which stands for Common Intermediate Language. This is read by the CLR, the Common Language Runtime. The CLR is a platform specific implementation, which generates machine code readable on the current  platform.

### Ok but what what differentiates platforms?

This is a great question. I mean, do I need different binaries for different versions of Windows? Do I need to have a different binary if I want to run my program on Windows 11 running on IA32 (Intel/AMD processors) and when I want to run it on a shiny new Qualcomm Snapdragon X Elite laptop, which is an ARM processor? What about different types of IA32 processors, like a new shiny one or one which was made 10 years ago? What if I use the exact same hardware, but I use a completely different OS, like Ubuntu, which is Linux based? What if I don't have an OS? What if I don't have some libraries?

The answer is: it depends, but almost always. I won't go into the exact details, but basically 3 things should be the same:

1. **The binary should be compiled for the same set of instructions.**: This means that there should be no instructions in our binary, which the processor can not read. Or to be more exact, the microcode running on the processor (at least in the case of Intel/AMD IA32 based ones, I am not sure about other and older platforms). This is because binary instructions are something akin to a script, for the processor. The microcode, which is the CPU's firmware code basically reads these binary instructions, decodes them and decides how to control the CPU. It is fun to think about that unless your job is to write microcode, on these platforms you never actually touch the CPU hardware directly... ***BUT!*** If the software or hardware can map these calls from the other ISA to ours, we can run software made for other ISAs. This is called emulation. Depending on how complete the mapping (or faking) and how much is done in software (the less the better), we can have a seamless experience.
2. **The binary should be compiled for the same calling conventions.**: This means that when calling a function on the lowest level, there is a way we save our state and store/write parameters and return values into the registers of the CPU. This differs from hardware platform to OS. As this is defined well for existing platforms, there is no easy way to magically make this compatible. At the end of the day, this is always up to the engineers creating an OS (or freestanding binaries) to decide, but one most have a very good oversight and understanding regarding the instruction set to do this. At least for a given OS, this is constant for a given instruction set.
3. **The binary should call the same common resources**: This means that system calls should be the same. This is the main reason why we can't run a, for instance Windows binary without extra work on a Linux based OS. The binary calls Windows kernel (Win32) APIs, which are nonexistant for the Linux kernel. ***BUT!*** If the instruction set is compatible (both are for AMD64), then we can map these calls to Linux syscalls and because everything else is the same (well, we make sure the point 2. stands, by putting and reading stuff from the right place), we can run Windows executables on an other OS. Even in the same OS, if there is a reference to a different versioned library to be loaded and is not available, our executable will fail to run. A good example to this is when glibc is different on our computer than the one the program was compiled to load.

Most developers produce binaries for a given instruction set for a given software platform (OS). That is why you can run most downloaded executables on most computers and versions of Windows. Of course, some platforms like Apple are much less lenient with binary compaitbility, which is the UNIX philosophy afterall, because source compatibility is better then binary compatibility, from a system devlopment standpoint (meaning your built executables might not run on different versions of a software platform, but using the source code, you could compile it for newer versions without a hassle). Bot approaches have their ups and downs. From the user side Windows is easier, because you can use the same binary, but for the platform's developers', it is worse, because it gets harder and harder to manage newer and newer versions of the OS.

If you have used Gentoo/Slackware/Arch you might know that when building programs from source, you can choose the compilation target to be not generic but optimized for the given platform. This mostly reduces the output binary size and makes sure that it uses all, but only those features available for your current hardware platform. If you restore the system built like this to another computer with a different CPU or upgrade it, these binaries will NOT run. This is because each different model of CPU (no matter how little different it is), uses a different system of instructions on the very lowest level and will not run your code.

Combining all of these, we get something like Apple's Game Porting Toolkit, which can run Windows executables, made for AMD64/IA32 and using the Win32 API (expanded with others like DirectX), and can emulate the hardware on ARM chips of Apple and translate the Windows-specific API calls into MacOS syscalls. Similar to this is Box64 for instance on Linux on ARM.

**Basically, this problem is lifted if we write code which is interpreted/compiled on a common software runtime.** And while it really solves it (like JavaScript in the browser or C# withe the CLR), even in these scenarios, our code is bound to the software platform provided by this additional layer of software abstractions. It is just that we have less to worry about.

## What are the differences?

I will list the main differences you should think about. Of course, there might be additional ones, but in my opinion they can be categorized into one of these. (I am not counting new features, because of course new features will always happen in newer versions, duh.)

1. In .NET Core applications, you can pack the base runtime with the application. This means that it is a lot easier to make portable applications. In .NET Framework based applications, you have to have the .NET version installed on the host system. This also implies, that .NET is no longer a part of Windows as a component of the OS, but rather a freestanding entity.
2. .NET Core refactored many previously included modules, so the previous point can be made feasibly. This keeps the .NET system small enough to pack it up with the application. For more special needs previously included in the framework, they were made into NuGet packages, so developers can include them in their project easily when needed. NuGet is the official package distribution platform for .NET, similar to the Pyton Package Index.
3. .NET Core is open source and multiplatform. You can utilize the .NET Core runtime on basically any platform: Windows, Linux distros, BSD distros, Android, IOS, MacOS, etc. You can also create freestanding binaries, which run on microcontrollers even! The Core SDK also provides the `dotnet` command, which provides a command line interface for any operation you would need regarding development, eleminating the need to use Visual Studio (for most of the time). Open sourcing includes all of the GUI frameworks in .NET Framework: WPF, WinForms, Windows UI and Platform Extensions (Take note that these are only available in Core since Core 3.0.)

## So which one should I use?

If you are starting a brand new project, use Core in every case basically! Not only that, try to use the most up-to-date version as possible! By this, you can use the newest feature level of C# and modern, forward thinking features like [MAUI](https://dotnet.microsoft.com/en-us/apps/maui) to build cross-platform applications and reduce the workload on your team.

Of course, if you have a legacy project, stay at that version and make sure to upgrade for security versions.

If you seriously want to port your codebase (but at that point, just create a new one), use [try-convert](https://github.com/dotnet/try-convert), which might solve it. (Note that, the repo is archived and I am not sure if the functionality is still valid.)
If your application relies or certain technologies, like application domains and technologies based on that like remoting through application domains, there is no direct equivalent to it and it is a breaking change. Also, custom build processes might not work in Core.

As always, my best advice is that if you want to have official support, consult the [.NET Support Policy page](https://dotnet.microsoft.com/en-us/platform/support/policy).

## What's next

In the next blogpost I plan to write up something about Entity Framework, which is one of the main reasons I like working with C#. See you there!
