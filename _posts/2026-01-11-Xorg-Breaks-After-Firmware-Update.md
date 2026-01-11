---
layout: post
title: Xorg Breaks After Firmware Update
categories: linux
author: avnogy
tags:
  - linux
  - xorg
  - firmware
  - troubleshooting
---

After updating UEFI on my desktop PC I tried to start my X Session and got the following error:

```
[   499.820] (EE) No devices detected.
[   499.820] (EE)
Fatal server error:
[   499.820] (EE) no screens found(EE)
[   499.820] (EE)
Please consult the The X.Org Foundation support
	 at http://wiki.x.org
 for help.
[   499.820] (EE) Please also check the log file at "/var/log/Xorg.0.log" for additional information.
[   499.820] (EE)
[   499.887] (EE) Server terminated with error (1). Closing log file.
```

I upgraded all the packages, hoping to solve any driver compatibility issues since I hadn’t updated the UEFI in a while, but the X session still failed to start.

looking at `/etc/X11/xorg.conf`:

```sh
$ cat /etc/X11/xorg.conf
# Omitted for readability ...

Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BusID          "PCI:5:0:0"
EndSection
```

then at the output of `lspci`:

```sh
$ lspci | grep -i nvidia
02:00.0 VGA compatible controller: NVIDIA Corporation GA104 [GeForce RTX 3060 Ti] (rev a1)
02:00.1 Audio device: NVIDIA Corporation GA104 High Definition Audio Controller (rev a1)
```

do you spot the difference?

Xorg believes the screen is at `PCI:5:0:0` but in reality it's at `PCI:02:00.0`! The firmware update reordered the devices..

![](/notes/assets/gifs/cmon.gif){: width="300" }

## How to Fix

Let's move the old configuration and create a new one:

```sh
mv /etc/X11/xorg.conf /etc/X11/xorg.conf.old
sudo nvidia-xconfig
```

This is a good sign:

```diff
$ diff /etc/X11/xorg.conf /etc/X11/xorg.conf.old
- BusID          "PCI:5:0:0"
+ BusID          "PCI:2:0:0"
```

We got an X session working!!

## What if We Don't Specify a BusID

Interestingly, `nvidia-xconfig` assigns a PCI slot for its screen, but we dont have to since we have only one screen compatible with Nvidia, so it's unique enough:

```diff
Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
-    BusID          "PCI:2:0:0"
EndSection
```

Still works!
