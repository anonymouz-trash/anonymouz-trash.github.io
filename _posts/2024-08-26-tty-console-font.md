---
layout: post
title:  "Configuring the TTY console font"
categories: [Client Administration]
tags: [arch,linux,client,administration,tty,console,font]
image:
  path: /assets/img/2024-12-20-tty-console-font.jpg
#last_modified_at: 2024-08-26 10:38:00 +0100
---

This is for the people that are old ;-), like me,  or with 4K displays.

## Font location
On Debian systems it should be: `/usr/share/consolefonts`
On Arch Linux systems it should be: `/usr/share/kbd/consolefonts`

I'm always looking for something like this: `UTF-8, Latin 1, Terminus, 14x28`

## Installation
In case Terminus font is not installed.

### Debian
```bash
sudo apt install xfonts-terminus
```

### Arch Linux
```bash
sudo pacman -S terminus-font
```

## Configuration

### Debian
```bash
setfont /usr/share/consolefonts/Lat15-Terminus28x14.psf.gz
```
Permanent:
```bash
dpkg-reconfigure console-setup
```
Or edit `/etc/default/console-setup` and restart.
```bash
CHARMAP="UTF-8"
CODESET="Lat15"
FONTFACE="Terminus"
FONTSIZE="14x28"
```
{: file="/etc/default/console-setup"}

### Arch Linux
```bash
setfont ter-132n
```
Permanent:

Edit `/etc/vconsole.conf`.
```bash
KEYMAP=de
FONT=ter-132n
```
{: file="/etc/vconsole.conf"}

Then restart systemd-service for vconsole.
```bash
sudo systemctl restart systemd-vconsole-setup.service
```
And restart for changes to take effect.
