---
layout: post
title:  "Schedule jobs with systemd-timer"
categories: [Server Administration]
tags: [debian,linux,server,administration,schedule,tasks,systemd,timer,cron]
image:
  path: /assets/img/2024-12-22-schedule-jobs.jpg
last_modified_at: 2024-10-14 06:34:00 +0100
---
Since I had trouble using cron, and crontab, in the past I decided to use `systemd` instead. You are also able to monitor them in a better way than in `cron`.

## 1st Example: CrowdSec hub update in docker container
Hub updates in the CrowdSec container must be handled manually because there is no (systemd) services installed which can handle that.   
At first I created a basic shell script, for instance, called `crowdsec-hub-update`. Yes, without `.sh` file extension.
```bash
#!/usr/bin/bash
docker exec crowdsec cscli hub update
docker exec crowdsec cscli hub upgrade
```
{: file="/usr/local/bin/crowdsec-hub-update"}
> Hint: Create this file in `/usr/local/bin` to make it globally available.
{: .prompt-info }

### Configuration
Create a timer-file in `/etc/systemd/system` called `crowdsec-hub-update.timer`.
```bash
[Unit]
Description="CrowdSec hub update everyday at every hour."
[Timer]
Unit=crowdsec-hub-update.service
OnBootSec=1min
OnCalendar=*-*-* *:00:00 # DOW YYYY-MM-DD HH:MM:SS
[Install]
WantedBy=timers.target
```
{: file="/etc/systemd/system/crowdsec-hub-update.timer"}

| value | description |
|---|---|
| Unit= | which unit (or service-file) this timer triggers |
| OnBootSec= | waiting after boot before 1st executing, comment or delete this line if you just want to make use of `OnCalendar` entry |
| OnCalendar= | defines a exact time, like in cron, when the unit is executed next |

After that create the corresponding unit in the same folder called `crowdsec-hub-update.service`.
```bash
[Unit]
Description="Update and upgrade CrowdSec Hub."
Requires=crowdsec-hub-update.timer
[Service]
Type=simple
ExecStart=/path/to/crowdsec-hub-update
User=root
```
{: file="/etc/systemd/system/crowdsec-hub-update.service"}

| value | description |
| --- | --- |
| Requires= | On which object does it depend |
| ExecStart= | path to the script, this is handy if you have more than one command |
| User= | which user executes the command |

### Usage
```bash
systemctl enable crowdsec-hub-update.timer
systemctl ( start | stop ) crowdsec-hub-update.timer
```
> When you start the timer the service will be executed once!
{: .prompt-warning}

## 2nd Example: Add MACVLAN interface for AdGuard Home container
I want my AdGuard Home container to have its own IP address, but this setting, however, gets lost on every reboot.
So...
I created a bash script named `add-macvlan-interface`.
```bash
#!/bin/bash
ip link add mac0 link eth0 type macvlan mode bridge
ip addr add 192.168.0.252/30 dev mac0
ifconfig mac0 up
```
{: file="add-macvlan-interface"}
> This creates a new device called `mac0` linked as bridge to `eth0` with an gateway IP set. When the docker container boots, it is a member of this network, the network interface will obtain the first ip which is `.253`.
{: .prompt-info }

### Configuration tasks at boot
Create a service-file in `/etc/systemd/system` named, e.g., `add-mac-vlan-int.service`.
```bash
[Unit]
Description="Add Docker MacVLAN interface for AdGuard Home."
Wants=network.target
After=syslog.target network-online.target
[Service]
Type=simple
ExecStart=/home/dietpi/git/add-macvlan-interface
Restart=on-failure
RestartSec=10
KillMode=process
User=root
[Install]
WantedBy=multi-user.target
```
{: file="/etc/systemd/system/add-mac-vlan-int.service"}
> It is the same process as creating recurring tasks, but without a timer-file. This unit will start at boot (when you enable it) after the network got initialized.
{: .prompt-info }

### Usage

```bash
systemctl enable add-macvlan-int.service
```
> You can add `--now` if you want to also start the service with enabling.
{: .prompt-info }

## Troubleshooting
```bash
systemctl status crowdsec-hub-update.timer
systemctl status crowdsec-hub-update.service
```
```bash
jounralctl -u crowdsec-hub-update.service
```
