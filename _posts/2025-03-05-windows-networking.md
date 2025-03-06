---
layout: post
title:  "Windows Networking"
categories: [Server Administration]
tags: [debian,linux,server,administration,network,interface]
image:
  path: /assets/img/2025-03-05-windows-networking.jpg
last_modified_at: 2025-03-06 07:45:00 +0100
---
In this post I describe simple network configuration steps of Windows servers and some commands for troubleshooting and testing. This all happens in PowerShell (v. 7.5), of course. :-) 

## Gather network adapters information
```shell
Get-NetAdapter
```
Output:
```console
Name                      InterfaceDescription                    ifIndex Status       MacAddress             LinkSpeed
----                      --------------------                    ------- ------       ----------             ---------
Ethernet-Instanz 0        Red Hat VirtIO Ethernet Adapter              12 Up           52-54-00-43-E3-00        10 Gbps
```
> Get the `ifIndex` number of our interface.
{: .prompt-info }

```shell
Get-NetIPConfiguration
```
```console
InterfaceAlias       : Ethernet-Instanz 0
InterfaceIndex       : 12
InterfaceDescription : Red Hat VirtIO Ethernet Adapter
NetProfile.Name      : Netzwerk 2
IPv4Address          : 192.168.122.23
IPv4DefaultGateway   : 192.168.122.1
DNSServer            : 9.9.9.9
                       149.112.112.112
```
> Get ip configuration, if there's any.
{: .prompt-info }

```shell
Get-DnsClientServerAddress
```
```console
InterfaceAlias               Interface Address ServerAddresses
                             Index     Family
--------------               --------- ------- ---------------
Ethernet-Instanz 0                  12 IPv4    {9.9.9.9, 149.112.112.112}
Ethernet-Instanz 0                  12 IPv6    {}
Loopback Pseudo-Interface 1          1 IPv4    {}
Loopback Pseudo-Interface 1          1 IPv6    {fec0:0:0:ffff::1, fec0:0:0:ffff::2, fec0:0:0:ffff::3}
```
> Get DNS server info, if there's any.
{: .prompt-info }

```shell
Get-NetIPInterface
```
```console
ifIndex InterfaceAlias                  AddressFamily NlMtu(Bytes) InterfaceMetric Dhcp     ConnectionState PolicyStore
------- --------------                  ------------- ------------ --------------- ----     --------------- -----------
1       Loopback Pseudo-Interface 1     IPv6            4294967295              75 Disabled Connected       ActiveStore
12      Ethernet-Instanz 0              IPv4                  1500              15 Enabled  Connected       ActiveStore
1       Loopback Pseudo-Interface 1     IPv4            4294967295              75 Disabled Connected       ActiveStore
```
> Check the `ConnectionState` and if the interface is configured to get its IP configuration from a `Dhcp`-Server.
{: .prompt-info }

## Configure static ip-address
```shell
# Turn off DHCP on the desired interface
Set-NeTIPInterface -InterfaceIndex 12 -Dhcp Disabled

# Set a new static IP address
New-NetIPAddress -InterfaceIndex 12 -AddressFamily IPv4 -IPAddress 10.0.1.4 -PrefixLength 27 -DefaultGateway 10.0.1.1

# Set DNS server info
Set-DNSClientServerAddress -InterfaceIndex 12 -ServerAddresses 9.9.9.9, 1.1.1.1
```

In case you mistyped the new configuration you are able to delete the IP address.
```shell
Remove-NetIPAddress -InterfaceIndex 12 -IPAddress 10.0.1.4
Remove-DNSClientServerAddress -InterfaceIndex 12 -ServerAddresses 9.9.9.9, 1.1.1.1
```

Afterwards test your settings with the CmdLets above.

## Troubleshooting

Deactivate a network adapter
```shell
Get-NetAdapter -InterfaceIndex 12 | Disable-NetAdapter
```
or
```shell
Disable-NetAdapter "*"
```
to deactivate all network adapters.

Use the opposite to `Enable-NetAdapter` again.

## Show listinening (open) ports on the server

```shell
Get-NetTCPConnection -State Listen,Established
```
