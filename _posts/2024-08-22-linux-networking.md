---
layout: post
title:  "Linux Networking"
categories: [Server Administration]
tags: [debian,linux,server,administration,network,interface]
image:
  path: /assets/img/2024-12-22-networking.jpg
last_modified_at: 2024-08-26 10:57:00 +0100
---
In this post I describe simple network configuration steps of Linux servers and some commands for troubleshooting and testing.

## Get network adapters
```bash
ip -c a
```
Output should look like:
```terminal
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 0c:9d:92:11:22:33 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.16/24 brd 192.168.0.255 scope global dynamic noprefixroute eth0
       valid_lft 730sec preferred_lft 730sec
    inet6 fe80::1e2a:5533/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
> Now you know that your network adapter is *eth0*. Depending on the distribution you use other names like *enp0s1* are also possible.
{: .prompt-info }

## Configure static ip-address
Edit `/etc/network/interfaces`.
```bash
auto eth0
iface eth0 inet static
 address 192.168.0.4
 netmask 255.255.255.0
 gateway 192.168.0.1
 dns-nameservers 192.168.0.1 100.100.100.100
```
{: file="/etc/network/interfaces"}

When finished restart network service.
```bash
systemctl restart networking
```
or
```bash
ifdown --all; ifup --all
```
or
```bash
ifdown eth0; ifup eth0
```

## Show listinening (open) ports on the server
```bash
ss -lntp
```
Output shoud look something like this:
```terminal
State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process                                 
LISTEN  0       32       192.168.122.1:53          0.0.0.0:*                                            
LISTEN  0       50                   *:1716              *:*     users:(("kdeconnectd",pid=1613,fd=21)) 
```
In this case, e.g. my linux client has an opend port for DNS (53) and 1716 which is used by KDEConnect.

#### Who's logged in right now?
```bash
w
```
Output looks like:
```terminal
user@server:~$ w
 13:00:46 up 6 days, 16:44,  1 user,  load average: 3.84, 2.82, 2.96
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
user1   pts/0    192.168.0.16     13:00    0.00s  0.09s  0.02s cat /etc/shadow
```
> Beside the logged in user you alse see the last executed command in the last row **(WHAT?!)**
{: .prompt-info }
