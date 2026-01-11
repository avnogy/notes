---
layout: post
title: How to install Nvidia Drivers on Debian the Right Way
categories: linux
author: avnogy
tags:
  - linux
  - nvidia
  - drivers
  - gpu
  - troubleshooting
---

AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

okay.

## What not to do

Initially, I visited the [official Nvidia site](https://www.nvidia.com/en-us/drivers/) to select my GPU.

![](/notes/assets/images/nvidia-installer.png)

I downloaded the official installer and followed the TUI installer. However, this method can lead to several issues, particularly with file overwrites of system distro packages (like `OpenGL`) that would just overwrite back after an upgrade.

## How to properly install Nvidia driver installations on Debian {#installation}

1. Add the repository from https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/debian.html#network-repository-enablement-amd64
2. Install via apt:

```bash
sudo apt install cuda-drivers
```

## Switching between driver versions

If you decide to switch between different driver versions (for example, from Nvidia's version to Debian's), ensure that no legacy files remain or everything will break and you will burn in hell forever.

1. Removing the old packages

First identify what you have:

```bash
dpkg -l | grep "ii .*nvidia"
```

Then remove the unwanted version

```bash
sudo apt-get remove $(dpkg -l | grep "ii .*nvidia-.*550" | awk '{print $2}')
```

2. Clearing outdated kernel drivers

I also found out there are kernel driver objects that stay if you use the official offline installer.

To clean them:

```bash
find /lib/modules/ -type f -name "nvidia.*"
```

From what I saw, `.deb` installations will end with .ko.xz, while installer-generated files may have the .ko extension. For instance, the location of my correct Nvidia-packaged modules often appears as follows:

```
/lib/modules/6.12.63+deb13-amd64/updates/dkms/nvidia.ko.xz
/lib/modules/6.12.57+deb13-amd64/updates/dkms/nvidia.ko.xz
```

Installer-generated modules are typically stored in a different directory than those from updates and DKMS.

When both driver versions are present, the system may prioritize the installer’s files. Just delete the ones you dont need and proceed to installing the right version

You can verify the active version being utilized with the following command after boot:

```bash
sudo dmesg | grep -i nvidia
```

---

After successfully removing the outdated packages, you can proceed to install the preferred Nvidia package according to [the installation section above](#installation).
