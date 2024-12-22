---
layout: post
title:  "Gamescope - Valve's upscaler"
categories: [Linux Gaming]
tags: [arch,linux,client,gaming,gamescope]
image:
  path: /assets/img/2024-12-22-gamescope.jpg
last_modified_at: 2024-08-26 10:56:00 +0100
---
Gamescope is a microcompositor from **Valve** that is used on the **Steam Deck**. Its goal is to provide an isolated compositor that is tailored towards gaming and supports many gaming-centric features such as:
* Spoofing resolutions
* Upscaling using AMD FidelityFX™ Super Resolution (FSR) or NVIDIA Image Scaling (NIS)
* Limiting framerates.

## Requirements
* AMD: Mesa 20.3 or above
* Intel: Mesa 21.2 or above
* NVIDIA: proprietary drivers 515.43.04 or above, and the nvidia-drm.modeset=1 kernel parameter

## Installation
```bash
sudo pacman -S gamescope
```

### Nvidia
Better understanding of how Nvidia DRM Modeset is designed found in [Nvidia forum](https://forums.developer.nvidia.com/t/understanding-nvidia-drm-modeset-1-nvidia-linux-driver-modesetting/204068/3) by an official called "aplattner".

> [...] All it really does is enable the DRIVER_MODESET capability flag in the nvidia-drm devices so that DRM clients can use the various modesetting APIs. In addition to allowing clients that talk to the low-level DRM interface to work, it’s also necessary for some PRIME-related interoperability features. [...]
{: .prompt-info }

To set it you have to add it `nvidia-drm.modeset=1` to the kernel (linux) command line in bootloaders, like for `GRUB` you have to edit `/etc/default/grub` as root.
```bash
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia-drm.modeset=1"
```
{: file="/etc/default/grub"}

Apply the changes and restart afterwards.
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
> For bootloaders other than GRUB checkout [Arch Wiki > Kernel paramters](https://wiki.archlinux.org/title/Kernel_parameters)
{: .prompt-tip }

## Usage
Gamescope offers many options, far too many to cover here. For a full list use the `gamescope --help` command from a terminal. 
```bash
gamescope -h 720 -H 1080 -F fsr -f --expose-wayland --
```
| parameter | description |
| --- | --- |
| -W, --output-width | output width (of display) |
| -H, --output-height | output height (of display) |
| -w, --nested-width | game width |
| -h, --nested-height | game height |
| -r, --nested-refresh | game refresh rate (frames per second) |
| -F, --filter | upscaler filter (linear, nearest, fsr, nis, pixel) |
| -b, --borderless | make the window borderless |
| -f, --fullscreen | make the window fullscreen |
| --expose-wayland | Allow support for wayland clients using xdg-shell |
> fsr => AMD FidelityFX™ Super Resolution 1.0
{: .prompt-info }

> nis => NVIDIA Image Scaling v1.0.3
{: .prompt-info }

### Steam
If you're using `Proton Experimental` or all other official builds then edit your start options per-game this way:
```bash
gamemoderun gamescope -h 720 -H 1080 -F fsr -f -- %command%
```

If you're using `Proton-GE` custom build then edit your start options per-game this way:
```bash
WINE_FULLSCREEN_FSR=1 WINE_FULLSCREEN_FSR_MODE=balanced WINE_FULLSCREEN_FSR_STRENGTH=1 %command%
```

### Any other lauchners
... have options for this, for example:
![gamescope-lutris](/assets/img/gamescope-lutris.png)
![gamescope-heroic](/assets/img/gamescope-heroic.png)
