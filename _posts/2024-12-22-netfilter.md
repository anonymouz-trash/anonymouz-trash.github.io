---
layout: post
title:  "Iptables / Nftables"
categories: [Server Administration]
tags: [debian,linux,server,administration,network,netfilter,iptables,nftables]
image:
  path: /assets/img/2024-12-22-netfilter.jpg
---
Iptables is the main packet filtering and firewalling tool in the Linux world. It's developed by a company named NetFilter. So iptables becomes deprecated over the time and nftables is the new implementation of iptables.

## Enable routing
With this command you can activate routing, but this setting is not perstistent and will get deactivated upon next reboot.
What happens is we change the kernel parameter for ip forwarding.
```bash
sysctl -w net.ipv4.ip_forward=1
```
To make this option persistent due to reboots you have to edit `/etc/sysctl.conf`. On Debian-like distros this option is already there and just has to be uncommented and changed to 1.
```bash
net.ipv4.ip_forward = 1
```
{: file="/etc/sysctl.conf"}

## Installation
```bash
apt install nftables
```

## Converting iptables to nftables
If you want to set something with iptables and try out rules.
```bash
iptables -t nat -A POSTROUTING  -s 192.168.2.0/24 -o enp0s3 -j MASQUERADE
```
> This command activates NAT-ing (Routing) on the "WAN"-Interface
{: .prompt-info }

```bash
nft list ruleset
```
```bash
table ip nat {
        chain POSTROUTING {
                type nat hook postrouting priority srcnat; policy accept;
                oifname "enp0s3" ip saddr { 192.168.2.0/24 } masquerade
        }
```
{: file="/etc/nftables.conf"}

> Then you have to copy the added line to `/etc/nftables.conf`
{: .prompt-info }

## Enable simple NAT-ing
Edit `/etc/nftables.conf'.
```bash
flush ruleset
...

table ip nat {
    chain prerouting {
        type nat hook prerouting priority 0; policy accept;
    }

    chain postrouting {
        type nat hook postrouting srcnat; policy accept;
        oifname "eth0" ip saddr { 192.168.0.0/24, 10.12.0.0/24 } masquerade
    }
}
```
{: file="/etc/nftables.conf"}

| value | description |
| --- | --- |
| oifname "eth0" | Specifies the WAN-Interface which is the NAT-Interface |
| ip saddr { } | You can leave this option you route all incoming traffic or specify a single ip net without brackets or multiple ip nets within brackets. |
| masquerade | This option must be set to swap the internal source ip address with the source ip address of the WAN/NAT interface |

## Troubleshooting
```bash
nft list ruleset
```
or
```bash
tcpdump -ni any icmp
```

| parameter | description |
| --- | --- |
| -n | No dns lookup |
| -i | interactive, lists actual traffic if there is any |
| any | Specifies the device, e.g. like eth0 |
| icmp | Specifies the protocol, e.g. like ping (icmp) for testing |

-----
> Further reading: [Gentoo nftables examples](https://wiki.gentoo.org/wiki/Nftables/Examples)
