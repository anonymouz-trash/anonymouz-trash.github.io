---
layout: post
title:  "Iptables / Nftables"
categories: [Server Administration]
tags: [debian,linux,server,administration,network,netfilter,iptables,nftables]
image:
  path: /assets/img/2024-12-22-netfilter.jpg
last_modified_at: 2025-06-07 08:09:00 +0100
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

## Use iptables as a script
For giving you an impression of how you can utilize iptables within a script with a few examples you can have a look at this script.

```shell
#!/bin/bash
# IP-Tables script
#
# program in short
ipt=$(which iptables)

# activate debugging aktivieren  'set -x'
#deactivate debugging 'set +x'

# Adapteraliases
green=enp7s0       # Client net
orange=enp8s0     # dmz
red=enp1s0            # internet

# Netze
green_net=10.10.11.0/24
dmz_net=10.10.21.0/24

### functions
function start(){
	# activate routing
	sysctl -w net.ipv4.ip_forward=1 > /dev/null
	echo "+ Routing enabled"

	# set implicit deny to block all traffic and whitelist allowed traffic afterwards
	${ipt} -P INPUT   DROP
	${ipt} -P OUTPUT  DROP
	${ipt} -P FORWARD DROP
	echo "+ IMPLICIT DENY"

	# clear all existing rules
	${ipt} -F INPUT
	${ipt} -F OUTPUT
	${ipt} -F FORWARD
	${ipt} -t nat -F
	${ipt} -t filter -F

	# activate stateful-inspection
	# when writing a rule in a specific direction then iptables automatically opens the way back
	${ipt} -I INPUT   -m state --state ESTABLISHED,RELATED -j ACCEPT
	${ipt} -I OUTPUT  -m state --state ESTABLISHED,RELATED -j ACCEPT
	${ipt} -I FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

	${ipt} -I INPUT  -i lo -j ACCEPT
	${ipt} -I OUTPUT -o lo -j ACCEPT
	echo "+ ACCEPT Loopback traffic"

		${ipt} -I OUTPUT -p icmp --icmp-type ping -j ACCEPT
	echo "+ Allow pinging from itself"

	${ipt} -A OUTPUT -p tcp --sport 1024: --dport 22 -j ACCEPT
	echo "+ Allow ssh to sshd port 22"

	${ipt} -A INPUT -p tcp -i ${green} --sport 1024: --dport 1337 -j ACCEPT
	echo "+ Allow ssh from green_net"

		${ipt} -A INPUT -p tcp -i ${red} --sport 1024: --dport 1337 -j ACCEPT
	echo "+ Allow ssh from red_net"

	${ipt} -A OUTPUT -p udp -o ${red} --sport 1024: --dport 53 -j ACCEPT
	${ipt} -A OUTPUT -p tcp -o ${red} --sport 1024: --dport 53 -j ACCEPT
	echo "+ Allow dns from internet"

	${ipt} -A INPUT -p udp -i ${green} --sport 1024: --dport 53 -j ACCEPT
	${ipt} -A INPUT -p tcp -i ${green} --sport 1024: --dport 53 -j ACCEPT
	echo "+ Allow dns from green_net"

	${ipt} -A INPUT -p udp -i ${orange} --sport 1024: --dport 53 -j ACCEPT
	${ipt} -A INPUT -p tcp -i ${orange} --sport 1024: --dport 53 -j ACCEPT
	echo "+ Allow dns from orange_net"

	${ipt} -A INPUT -p udp -i ${orange} --sport 123 --dport 123 -j ACCEPT
	${ipt} -A INPUT -p udp -i ${green} --sport 123 --dport 123 -j ACCEPT
	echo "+ Allow ntp from orange_net & green_net"

	${ipt} -A OUTPUT -p tcp -o ${red} --sport 1024: --dport 80 -j ACCEPT
	${ipt} -A OUTPUT -p tcp -o ${red} --sport 1024: --dport 443 -j ACCEPT
	echo "+ Allow https from internet"

	${ipt} -A OUTPUT -p udp -o ${red} --sport 123 --dport 123 -j ACCEPT
	echo "+ Allow ntp from internet"

	${ipt} -A INPUT -p tcp -i ${green} --sport 1024: --dport 3128 -j ACCEPT
	echo "+ Allow squid from green_net"

	${ipt} -A INPUT -p tcp -i ${orange} --sport 1024: --dport 3128 -j ACCEPT
	echo "+ Allow squid from orange_net"

	# allow forwarding ping from green_net or orange_net to red_net
	${ipt} -A FORWARD -p icmp -i ${green} -o ${red} --icmp-type ping -j ACCEPT
	${ipt} -A FORWARD -p icmp -i ${orange} -o ${red} --icmp-type ping -j ACCEPT

	# ssh
	${ipt} -A FORWARD -p tcp -i ${green} -o ${red} --sport 1024: --dport 22 -j ACCEPT

	# ftp
	${ipt} -A FORWARD -p tcp -i ${green} -o ${red} --sport 1024: --dport 21 -j ACCEPT #ftp signal
	${ipt} -A FORWARD -p tcp -i ${green} -o ${red} --sport 1024: --dport 20 -j ACCEPT #ftp trsp

	# rdp
	${ipt} -A FORWARD -p tcp -i ${green} -o ${red} --sport 1024: --dport 3389 -j ACCEPT

	# smtp(s)
	${ipt} -A FORWARD -p tcp -i ${green} -o ${red} --sport 1024: --dport 25 -j ACCEPT
	${ipt} -A FORWARD -p tcp -i ${green} -o ${red} --sport 1024: --dport 465 -j ACCEPT
	echo "+ Appending forwarding rules from green to red (icmp,ssh,ftp,rdp,smtp)"

	# forwarding from red to orange

	# http
	${ipt} -A FORWARD -p tcp -i ${red} -o ${orange} --dport 80 -j ACCEPT
	#${ipt} -A FORWARD -p tcp -i ${red} -o ${orange} --dport 443 -j ACCEPT

	# smtp
	${ipt} -A FORWARD -p tcp -i ${red} -o ${orange} --dport 25 -j ACCEPT

	# ssh
	${ipt} -A FORWARD -p tcp -i ${red} -o ${orange} --dport 22 -j ACCEPT

	# DNAT für orange_net zu  dmz-srv
	${ipt} -t nat -A PREROUTING -p tcp -i ${red} --dport 80 -j DNAT --to 10.10.21.2
	${ipt} -t nat -A PREROUTING -p tcp -i ${red} --dport 25 -j DNAT --to 10.10.21.2
	${ipt} -t nat -A PREROUTING -p tcp -i ${red} --dport 3333 -j DNAT --to 10.10.21.2:22
	echo "+ Portforwarding :22,:25,:80 to 10.10.21.2"

	# Enable SNAT
	${ipt} -t nat -A POSTROUTING -o ${red} -j MASQUERADE
	echo "+ Enable NAT"

	# Activate Logging for INPUT and FORWARD
	${ipt} -A INPUT -j LOG --log-prefix "INPUT_LOG: "
	${ipt} -A FORWARD -j LOG --log-prefix "FORWARD_LOG: "
	echo "+ Logging enabled"
}

function stop(){
	sysctl -w net.ipv4.ip_forward=0 > /dev/null
	echo "+ Routing disabled"

	${ipt} -P INPUT   ACCEPT
	${ipt} -P OUTPUT  ACCEPT
	${ipt} -P FORWARD ACCEPT
	echo "+ ALLOW ANY ANY"

	${ipt} -F INPUT
	${ipt} -F OUTPUT
	${ipt} -F FORWARD
	${ipt} -t nat -F
	${ipt} -t filter -F
	echo "+ Tables and rules flushed"

	${ipt} -D INPUT -j LOG --log-prefix "INPUT_LOG: "
	${ipt} -D FORWARD -j LOG --log-prefix "FORWARD_LOG: "
	echo "+ Logging disabled"
}

function save_rules(){
	sudo iptables-save > /etc/iptables/rules.v4
	echo "Rules saved"
}

function status(){
	${ipt} -L -n -v
	${ipt} -t nat -L -n -v
}

# Bildschirm löschen
clear

case "$1" in
	start)
		start
		;;
	stop)
		stop
		;;
	restart)
		stop
		start
		;;
	status)
		status
		;;
	save)
		save_rules
		;;
	*)
		echo "Command: ${0} {start|stop|restart|status|save}"
esac
```
-----
> Further reading: [Gentoo nftables examples](https://wiki.gentoo.org/wiki/Nftables/Examples)
