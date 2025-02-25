---
layout: post
title:  "Start / Stop Wireguard VPN per script shortcut"
categories: [Linux Scripting]
tags: [linux,scripting,wireguard,vpn,shortcut]
image:
  path: /assets/img/2025-01-08-wg-vpn-script.jpg
last_modified_at: 2025-02-25 23:10:00 +0100
---

## Definition

This script is intended to use it as a keyboard shortcut and start / vpn the Wireguard VPN client. It also detects if the Wireguard connection is already configured or not.

When the script is started it checks for the Wireguard interface. If its there it stopps the client and removes the interface and if its not there its starting the client.

## Installation

* Copy your Wireguard VPN configuration in `/etc/wireguard` and name it `wg0.conf`.
* Save the script in `/usr/local/bin` to easily find it again and make it globally executable.
* Run `sudo chmod +x /usr/local/bin/wg-vpn` to make it executable (`wg-vpn` is just an example name)

> To display desktop notifications you'll have to install the `libnotify` package.
{: .prompt-tip}

## Script

```bash
#!/usr/bin/bash

# Specifiy the Wireguard interface (config) name
WG_INTERFACE_NAME=wg0

# First check if the output of 'ip link' contains the Wireguard interface
if [ $(ip l | grep "$WG_INTERFACE_NAME" | wc -l) -eq 0 ]; then
	# Wireguard gets started.
	# && means success notify
	# || means any error
	pkexec wg-quick up wg0 && notify-send -t 5000 -i "dialog-information" "Wireguard VPN" "$WG_INTERFACE_NAME is connected..." || notify-send -u critical -t 5000 -i "dialog-warning" "Wireguard VPN" "Something went wrong!"

else
  pkexec wg-quick down wg0 && notify-send -t 5000 -i "dialog-information" "Wireguard VPN" "$WG_INTERFACE_NAME is connected..." || notify-send -u critical -t 5000 -i "dialog-warning" "Wireguard VPN" "Something went wrong!"
fi
```
{: file="/usr/local/bin/wg-vpn"}

> Keep in mind that there is no check if your Wireguard VPN connection is actually established or not.
{: .prompt-warning}
