---
layout: post
title:  "Cloudflare DynDNS updater as Docker service"
categories: [Server Virtualization]
tags: [debian,linux,server,virtualization,docker,cloudflare,cf,ddns,dyndns,dns]
image:
  path: /assets/img/2024-12-22-docker-cf-ddns.jpg
last_modified_at: 2024-10-08 06:34:00 +0100
---
To update the IP of my domain I use this image which automatically queries the IP my provider assigned me and send it to Cloudflare if it has changed.
```yml
services:
  cloudflare-ddns:
    container_name: cloudflare-dyndns
    image: oznu/cloudflare-ddns:latest
    network_mode: bridge
    restart: unless-stopped
    environment:
      - API_KEY=${CF_DNSAPI}
      - ZONE=${CF_Zone}
      - RRTYPE=A
      - PROXIED=false
```
{: file="cf-ddns-updater/compose.yml"}
> To make use of `${variable}` you have to create a `.env` file in the same folder and add all your variables like `variable=value`
{: .prompt-tip }
