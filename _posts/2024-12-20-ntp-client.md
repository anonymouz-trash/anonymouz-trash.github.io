---
layout: post
title:  "Network Time Protocol (NTP)"
categories: [Client Administration]
tags: [arch,linux,client,administration,ntp,time]
image:
  path: /assets/img/2024-12-20-ntp.jpg
---

## Definition

Usually there is no need to install any package if you're using `systemd`. Then you have `systemd-timesyncd`.

## Configuration
Edit `/etc/systemd/timesyncd.conf`.
```bash
[Time]
NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
FallbackNTP=0.pool.ntp.org 1.pool.ntp.org 0.fr.pool.ntp.org
[...]
```
{: file="/etc/systemd/timesyncd.conf"}

To check your configuration you can use:
```bash
timedatectl show-timesync --all
```

Output should look like:
```terminal
LinkNTPServers=
SystemNTPServers=
FallbackNTPServers=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
ServerName=0.arch.pool.ntp.org
ServerAddress=103.47.76.177
RootDistanceMaxUSec=5s
PollIntervalMinUSec=32s
PollIntervalMaxUSec=34min 8s
PollIntervalUSec=1min 4s
NTPMessage={ Leap=0, Version=4, Mode=4, Stratum=2, Precision=-21, RootDelay=177.398ms, RootDispersion=142.196ms, Reference=C342F10A, OriginateTimestamp=Mon 2018-07-16 13:53:43 +08, ReceiveTimestamp=Mon 2018-07-16 13:53:43 +08, TransmitTimestamp=Mon 2018-07-16 13:53:43 +08, DestinationTimestamp=Mon 2018-07-16 13:53:43 +08, Ignored=no PacketCount=1, Jitter=0 }
Frequency=22520548
```

## Usage
To enable/disable NTP:
```bash
timedatectl set-ntp true
```

To check the service:
```bash
timedatectl status
```

Output:
```terminal
               Local time: Thu 2015-07-09 18:21:33 CEST
           Universal time: Thu 2015-07-09 16:21:33 UTC
                 RTC time: Thu 2015-07-09 16:21:33
                Time zone: Europe/Amsterdam (CEST, +0200)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

## Troubleshooting
The service writes to a local file `/var/lib/systemd/timesync/clock` with every synchronization and every 60 seconds. This location is hard-coded and cannot be changed. To view this log type the following.
```bash
journalctl -u systemd-timesyncd --no-hostname --since "1 day ago"
```

Output is like:
```terminal
Jan 19 15:14:20 systemd[1]: Stopping Network Time Synchronization...
Jan 19 15:14:20 systemd[1]: systemd-timesyncd.service: Deactivated successfully.
Jan 19 15:14:20 systemd[1]: Stopped Network Time Synchronization.
Jan 19 15:14:20 systemd[1]: Starting Network Time Synchronization...
Jan 19 15:14:20 systemd[1]: Started Network Time Synchronization.
Jan 19 15:14:20 systemd-timesyncd[1023]: Contacted time server 178.215.228.24:123 (0.nl.pool.ntp.org).
Jan 19 15:14:20 systemd-timesyncd[1023]: Initial clock synchronization to Fri 2024-01-19 15:14:20.393865 CET.
```

A more verbose status:
```bash
timedatectl timesync-status
```
```terminal
       Server: 103.47.76.177 (0.arch.pool.ntp.org)
Poll interval: 2min 8s (min: 32s; max 34min 8s)
         Leap: normal
      Version: 4
      Stratum: 2
    Reference: C342F10A
    Precision: 1us (-21)
Root distance: 231.856ms (max: 5s)
       Offset: -19.428ms
        Delay: 36.717ms
       Jitter: 7.343ms
 Packet count: 2
    Frequency: +267.747ppm
```
