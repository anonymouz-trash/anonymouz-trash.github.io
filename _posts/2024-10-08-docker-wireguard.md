---
layout: post
title:  "Install & configure Wireguard in Docker"
categories: [Server Virtualization]
tags: [debian,linux,server,virtualization,docker,wireguard,vpn]
image:
  path: /assets/img/2024-12-22-docker-wireguard.jpg
last_modified_at: 2024-10-09 06:14:00 +0100
---
Your very own VPN-service and it's also linked to AdGuard Home. ;o)
```yml
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE #optional
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
      - SERVERURL=${SERVERURL}
      - SERVERPORT=${SERVERPORT}
      - PEERS=${PEERS}
      - PEERDNS=192.168.1.253 #optional
      - INTERNAL_SUBNET=10.13.13.0 #optional
      - ALLOWEDIPS=0.0.0.0/0 #optional
      - LOG_CONFS=true #optional
    volumes:
      - ./config:/config
      - /lib/modules:/lib/modules #optional
    ports:
      - 51822:51820/udp
    networks:
      adguard_default:
        ipv4_address: 172.30.0.3 # fixed IP address
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
networks:
  adguard_default:
    external: true
```
{: file="wireguard/compose.yml"}

## Usage
If you want to show a QR-Code for specific peer in CLI then:
```bash
docker exec -it wireguard /app/show-peer <PEER>
```
