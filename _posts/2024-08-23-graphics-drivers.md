---
layout: post
title:  "Linux graphics driver setup"
categories: [Linux Gaming]
tags: [arch,linux,client,gaming,driver,nvidia,amd,intel]
image:
  path: /assets/img/2024-12-22-graphics-drivers.jpg
last_modified_at: 2024-08-25 07:52:00 +0100
---
## Prerequisites
First, enable multilib (32-Bit). It is very important to run games through Proton (Steam) and other WINE-related applications like Lutris, Bottles and Heroic Game Launcher.
To enable multilib repository, uncomment the `[multilib]` section in `/etc/pacman.conf`.
```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```
{: file="/etc/pacman.conf"}

Then upgrade your system.
```bash
sudo pacman -Syu
```

## Installation
To install support for Vulkan API (will be functional only if you have a Vulkan capable GPU) and 32-bit games, execute following commands.

### Nvidia
```bash
sudo pacman -S --needed nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader
```
> **Warning:** Installing *nvidia-settings* on Manjaro will fail as it isn't in the repos and gets installed with the drivers themselves. To make sure you're running proprietary Nvidia drivers you should run `sudo mhwd -i pci video-nvidia`
{: .prompt-warning }

I use the *nvidia-dkms* package to let Nvidia compile the driver against the actual kernel(s) I run. The other packages like `nvidia` or `nvidia-lts` are precompiled and may not suit the kernel(s) running.

### AMD
```bash
sudo pacman -S --needed lib32-mesa vulkan-radeon lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader
```

### Intel
```bash
sudo pacman -S --needed lib32-mesa vulkan-intel lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader
```
> Note for Intel integrated graphics users: Only Skylake and newer Intel CPUs (processors) offer full Vulkan support. Broadwell, Haswell and Ivy Bridge only offer partial support, which will very likely not work properly with a lot of games. Sandy Bridge and older lack any Vulkan support whatsoever.
{: .prompt-info }

## Preparing the kernel
It's recommended to add all needed driver modules to `initial ramdisk` to include the modules (drivers) into the kernel when it's booting.
To achieve this add the modules to `/etc/mkinitcpio.conf` and recompile `initramfs`.
```bash
MODULES=(i915 nvidia nvidia_modeset nvidia_uvm nvidia_drm)
BINARIES=()
FILES=()
HOOKS=(base udev plymouth autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```
{: file="/etc/mkinitcpio.conf"}

> For me, I have a laptop with switchable graphics, and therefore I would also load `i915` for the Intel card besides the Nvidia modules. For AMD users it would be `amdgpu`. More of that in the "Hybrid Systems"-Section.
{: .prompt-info }

Once you're finished, `initramfs` is recompiled like this:
```bash
sudo mkinitcpio -P linux
```
> For `linux` you have to set your wanted kernel image, others could be `linux-lts` or `linux-zen`.
{: .prompt-info }

## Hint: Missing firmware messages
If you get missing firmware messages as shown below, there's the [`mkinitcpio-firmware` AUR package](https://aur.archlinux.org/packages/mkinitcpio-firmware) to get rid of them.
```terminal
==> WARNING: Possibly missing firmware for module: bfa
==> WARNING: Possibly missing firmware for module: qed
==> WARNING: Possibly missing firmware for module: qla1280
==> WARNING: Possibly missing firmware for module: qla2xxx
```

Install it with an AUR helper like yay:
```bash
yay -S mkinitcpio-firmware
```
> For most users these messages are harmless, and "Possibly" means that you might not actually need this firmware. Anyway, if these messages are annoying you, you can work around it this way.
{: .prompt-info}

-----
Further reading: [Lutris docs > Installing Drivers](https://github.com/lutris/docs/blob/master/InstallingDrivers.md)
