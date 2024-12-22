---
layout: post
title:  "Install & configure Gluetun in Docker"
categories: [Server Virtualization]
tags: [debian,linux,server,virtualization,docker,gluetun,vpn]
image:
  path: /assets/img/2024-12-22-docker-gluetun.jpg
last_modified_at: 2024-10-08 06:57:00 +0100
---
Very cool service to add a VPN connection to your other services if needed. Downloaders maybe a good example. ;o)
```yml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    #    privileged: true
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    networks:
      - proxy
    ports:
      # - 8888:8888/tcp # HTTP proxy
      # - 8388:8388/tcp # Shadowsocks
      # - 8388:8388/udp # ShadowsocksShadowsocks
      - 8000:8000/tcp # http control server
      - 5959:8080 # sabnzbd
      - 8112:8112 # deluge
      - 6881:6881
      - 6881:6881/udp
    volumes:
      - ./config:/gluetun
    environment:
      # See https://github.com/qdm12/gluetun-wiki/tree/main/setup#setup
      - VPN_SERVICE_PROVIDER=${provider}
      - OPENVPN_USER=${OVPN_USER}
      - OPENVPN_PASSWORD=${OVPN_PASS}
      - SERVER_COUNTRIES=${Country}
      - TZ=${TZ}
      - BLOCK_MALICIOUS=on
      - BLOCK_SURVEILLANCE=on
      - FIREWALL_OUTBOUND_SUBNETS=192.168.1.0/24,172.25.0.0/16
    restart: always
networks:
  proxy:
    driver: bridge
    external: true
```
{: file="gluetun/compose.yml"}

> FIREWALL_OUTBOUND_SUBNETS = Whitelisted local networks. See [https://github.com/qdm12/gluetun-wiki/tree/main/setup#setup](https://github.com/qdm12/gluetun-wiki/tree/main/setup#setup) to configuarate your VPN-provider.
{: .prompt-tip }

> You must remove the `ports`-section of every container attached to Gluetun.
{: .prompt-danger }

## Usage
On every container you want to attach you have to set `-n container:gluetun` or `network-mode: container:gluetun` if you use compose.
