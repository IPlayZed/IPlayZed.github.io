+++
title = "How to enable UEFI and secure boot when running Linux in VmWare Workstation Pro"
date = 2024-08-01
updated = 2024-08-01
description = ""
[taxonomies]
tags = ["linux", "virtualization", "vmware", "secure boot", "uefi", "security", "windows", "vmware"]
+++

## The prologue

Recently I moved to backed to Windows 11, due to my curiosity of state of the things (the grass if always greener on the neighbour's side I guess 🙄) and because I started working on some game decompilation stuff and using Wine for it is not as comfortable as I would like it to be.

Still, I do have plans (and all my backups actually from my Arch installations) to do several small projects for Linux, one of which is a Arch Linux ALPM hook manager application using [MAUI and C#](https://dotnet.microsoft.com/en-us/apps/maui). (Yes, C# has been open source and cross-platform since .NET Core. It is even usable since .NET 6.) I got into writing my own hooks and realized that basically there is no good guide on how to put them together, but they are ridicoulosly powerful things. One of the ideas I have learnt from MacOS (yes, more on that later, because I started doing dev stuff there as well), is that you can utilize flowchart based GUI applications to kinda drag and drop very complex automation together.

This might sound bad for a seasoned programmer (and I do prefer just scripting things or writing my SystemD service files for it), but looking into it deeper, most non-software-engineer people working with computers prefer the visual method. Think about complex tools like [Node-RED](https://nodered.org/ "the Node-RED website"), [LabVIEW](https://www.ni.com/en/shop/labview.html "the LabVIEW website") or one of the software I was developing when working at Evosoft: [TIA Portal](https://www.siemens.com/global/en/products/automation/industry-software/automation-software/tia-portal.html "a tool for everything engineering").

Long story short, I wanted to find a good performance VM to develop my app, and while WSL [does support GUI apps](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps), I was not sure if I could relaiably debug them or if I would run into problems, which only exist because of WSL. I plan to make a post about the process of trying out other virtualization platforms avaiable on Windows and which ones had what kind of problems.

Of course the first thing I do is that I try to install the latest Ubuntu LTS for reference, as it is the most likely to play well on anything and everything.

## Getting VMWare Workstation Pro

According to my tests, the easiest and best experience on Windows 11 was using VMWare's virtualization technology.

Recently, [VMWare was bought by Broadcom](https://investors.broadcom.com/news-releases/news-release-details/broadcom-completes-acquisition-vmware). For us, developers, this means two things:

* you will have to create a Broadcom account to download VMWare software,
* and by extension, seems like VMWare now can afford to offer Workstation Pro (17 or newer for Windows/Linux) and Fusion Pro (13 or newer for MacOS) to be freely used by individuals, yipee!

I do know that some folks concerned with their privacy will not be happy about Broadcom tracking their software downloads and usage, but at least we can get the very best legally, for free.

One thing to note is that they seemingly forgot to update their download infrastructure and links or most search engines do not show the correct pages. The [evaluation download website](https://www.vmware.com/info/workstation-player/evaluation) seems to be up and running, but clicking on the download link under *Try Workstation 17 Player for Windows* (or the Linux link), will result in a Cloudflare error of 522, which is thrown when the host bridged by Cloudflare is down.

A good guide was written by [Mike Roy](https://www.mikeroysoft.com/post/download-fusion-ws/), if you want to follow that. Basically you have to register on Broadcom's website and use their download portal there do get the installer.

## Creating the VM correctly

**Note: This applies to 17.5.2 build-23775571 on Windows 11. You might have a (slightly) different experience based on the build and platform used.**

Now, by default if you download the Ubuntu Desktop installer, VMWare will autodetect it being a Linux distribution.

We have to select UEFI under the VM's settings in the *Options* tab under *Advanced > Firmware type*. There is an ominous message about chaning the firmware interface making the VM unbootable. This is kind of true. When the OS installer installs the OS, it configures the bootloader in a way that is conforms to IBM's BIOS heresay "standards" or the new UEFI specs. Technically you could convert any system to another (except newer Windows versions, because Windows' *bootmgr.efi* (the loader for the Windows kernel) only supports UEFI).

Because of the aforementioned reason you should only start the OS installation after this is changed to UEFI. Either decline starting the installation image when prompted at the end of the configuration or do not set any to begin with.

When selecting Windows 10 or newer installation image, a software TPM is created by the hypervisor for that specific virtual machine. It seems like, it is a requirement for the VMware hypervisor to create one and set it to enable the secure boot option in the previously mentioned *Firmware type* settings section. Logically, secure boot is a UEFI feature, so it is only available there.

There actually are similar attempts at making a similar functionality for non-UEFI systems, but they fail miserably in some way or are really inflexible. I have a Fujitsu based server ran for Nextcloud which has a feature like this, I will write about it in an other post as for it's age that workstation is very capable actually!

Creating a TPM manually can be done after the creation of the VM in the *Options* tab under *Access Control*. Not a very descriptive name, but it is basically a mandatory step to create the TPM, as it encrypts the virtual firmware as needed.

However, this is only available if the guest OS is set to Windows 10 or newer. So set it to that and you will be able to enable secure boot. It is really important to set these **before** installing the OS, as unless you are doing it manually in Arch for instance, the installer will assume that there is no secure boot/TPM/UEFI firmware, depending on which you have missed. In my tests, it works as intended and reliably. Compared to real hardware, I did not even get the annoying [TPM DA Lockout mode error](https://superuser.com/questions/1404738/tpm-2-0-hardware-error-da-lockout-mode) from the Ubuntu installer (which is just an inconvenience actually, as it can be [easily fixed](https://ubuntu.com/core/docs/troubleshooting#heading--tpm-lockout)).

## VMware has bugs and is wrong about things

Why do I say that? Well, for one, most Linux distributions support secure boot. To be more precise they are compatible with it and MOK system works well, but initramfs attacks are not secured against (unless you use [unified kernel images](https://uapi-group.org/specifications/specs/unified_kernel_image/), which is one executable launched by the firmware, which can be signed and checked against). But every distro you could think of supports UEFI nowadays through [GRUB](https://www.gnu.org/software/grub/)/[systemd-boot](https://systemd.io/BOOT/) and you can even boot the kernel directly through [EFISTUB](https://wiki.archlinux.org/title/EFISTUB)!

So VMware setting the firmware type to BIOS for linux guests is utter nonsense and is probably a bad piece of unrefactored code left in their codebase to be honest.

Also, if you set the guest OS type to Windows, save the settings and try setting it to a Linux variant, you will run into the error message: *This VM is 'UEFI' enabled, but you are trying to set the VM as a Guest OS that does not support 'UEFI'. Discard your changes or use a valid Guest OS Version.*

Well duh, this is nonsense. What is more - even though it does defult to BIOS -, you can set UEFI as the firmware platform even for Linux guests after the creation of the VM. Maybe I will try to make a bug report for these things.

## Addendum

Did you guys enjoy this post? Did you manage to learn something useful? I am looking for any and all feedback and ideas, please hit me up on any of the contacts by clicking the contact page or the social icons at the bottom of the page.

See you in the next post!
{{ gif(sources=["https://tenor.com/view/peach-and-goma-gif-1151492026803486934"]) }}
