---
layout: post
title:  "Install & configure AdGuard Home in Docker"
categories: [Server Virtualization]
tags: [debian,linux,server,virtualization,docker,adguard,home]
image:
  path: /assets/img/2024-12-22-docker-adguard.jpg
last_modified_at: 2024-10-08 06:57:00 +0100
---
In this example AdGuard Home uses its own IP-Adress but is on the same server as the other services. To achieve this you have to create a so called `macvlan` network / interface. It also uses `Unbound` to query all adresses top-down from the root DNS servers instead of asking the next ones. Those requests maybe slower the first time but even faster when cached.
```yml
services:
  adguardhome:
    depends_on:
      - unbound
    image: adguard/adguardhome:latest
    container_name: adguardhome
    #    ports:
    #      - 53:53/tcp # DNS
    #      - 53:53/udp
    #      - 784:784/udp # DNS over QUIC
    #      - 853:853/tcp # DNS over TLS
    #      - 1212:3000/tcp # initial installation
    #      - 1313:80/tcp # Dashboard
    #      - 4443:443/tcp # DNS over HTTPs
    networks:
      default:
        ipv4_address: 172.30.0.2 # fixed IP address
      macvlan:
        ipv4_address: 192.168.1.253
    volumes:
      - ./config/work:/opt/adguardhome/work
      - ./config/conf:/opt/adguardhome/conf
      - ./config/sysctl.conf:/etc/sysctl.conf
    restart: unless-stopped
  unbound:
    container_name: unbound
    restart: unless-stopped
    image: klutchell/unbound:latest
    networks:
      default:
        ipv4_address: 172.30.0.53 # fixed IP address
networks:
  default:
    ipam:
      config:
        - subnet: 172.30.0.0/24
  macvlan:
    driver: macvlan
    driver_opts:
      parent: eth0
    ipam:
      driver: default
      config:
        - subnet: 192.168.1.0/24
          ip_range: 192.168.1.252/30
          gateway: 192.168.1.1
```
{: file="aduardhome/compose.yml"}

## MACVLAN-Interface on the server
If your MACVLAN interface gets deleted on restart you can use this script on autostart with cron or systemd to make it kind of persistent. You find an example [here](https://docs.skynetcloud.org/posts/schedule-jobs/#2nd-example-add-macvlan-interface-for-adguard-home-container).

```bash
#!/bin/bash
ip link add mac0 link eth0 type macvlan mode bridge
ip addr add 192.168.1.252/30 dev mac0
ifconfig mac0 up
```
{: file="add-macvlan-interface"}
