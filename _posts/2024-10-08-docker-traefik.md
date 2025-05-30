---
layout: post
title:  "Reverse Proxy, IDS/IPS & Single-Sign-On in Docker"
categories: [Server Virtualization]
tags: [debian,linux,server,virtualization,docker,reverse,proxy,ids,ips,traefik,crowdsec,authelia,sso]
image:
  path: /assets/img/2024-12-22-docker-traefik.jpg
last_modified_at: 2024-10-19 06:57:00 +0100
---

**Traefik** is a reverse proxy, but a router also. It redirects every wanted connection to your liking. This compose contains the needed configuration for CrowdSec Bouncer too and I will explain it later.

**CrowdSec** is a community-driven IDS/IPS-System. I will provide an easy to understand explaination later.

**Authelia** is a self-hosted Single-Sign-On service to protect websites which don't have any form of authentication built-in.

I used this documentation site/video to achieve this:
{% include embed/youtube.html id='dgQvvMhbn8I' %}

> Written guide: [https://docs.ibracorp.io/crowdsec](https://docs.ibracorp.io/crowdsec)
{: .prompt-tip }

## CrowdSec & Traefik Bouncer
```yml
services:
  crowdsec:
    image: crowdsecurity/crowdsec:latest
    container_name: crowdsec
    environment:
      # In this section you can define your collections for parsing and handling log files for specific services
      CONFIGURATIONS: crowdsecurity/rdns
      COLLECTIONS: crowdsecurity/traefik LePresidente/authelia
        crowdsecurity/base-http-scenarios crowdsecurity/http-cve
        crowdsecurity/whitelist-good-actors crowdsecurity/http-dos 
        LePresidente/adguardhome 
      SCENARIOS: aidalinfo/tcpudp-flood-traefik
    volumes:
      - ./config/acquis.yaml:/etc/crowdsec/acquis.yaml
      - ./config/whitelists.yaml:/etc/crowdsec/parsers/s02-enrich/whitelists.yaml
      - ./config/fqdn-whitelist.yaml:/etc/crowdsec/postoverflows/s01-whitelist/fqdn-whitelist.yaml
      - crowdsec-db:/var/lib/crowdsec/data/
      - crowdsec-config:/etc/crowdsec/
      - /var/run/docker.sock:/var/run/docker.sock
      # According to the collections you have to allocate the corresponding log files
      - traefik_traefik-logs:/var/log/traefik/:ro
      - traefik_authelia-logs:/var/log/authelia/:ro
      - adguard_adguard-logs:/var/log/adguard/:ro
    networks:
      - proxy
    restart: unless-stopped
  bouncer-traefik:
    image: docker.io/fbonalair/traefik-crowdsec-bouncer:latest
    container_name: bouncer-traefik
    environment:
      CROWDSEC_BOUNCER_API_KEY: ${CROWDSEC_BOUNCER_API}
      CROWDSEC_AGENT_HOST: crowdsec:8080
    networks:
      - proxy # same network as traefik + crowdsec
    depends_on:
      - crowdsec
    restart: unless-stopped
networks:
  proxy:
    driver: bridge
    external: true
volumes:
  crowdsec-db: null
  crowdsec-config: null
  traefik_traefik-logs:
    external: true
  traefik_authelia-logs:
    external: true
  adguard_adguard-logs:
    external: true
```
{: file="traefik/compose.yml"}

## CrowdSec: acquis.yml
The acquisition file is very important to CrowdSec because it tells it where to find the logs.
```yml
filenames:
  - /var/log/traefik/*
labels:
  type: traefik
---
filenames:
 - /var/log/authelia/Authelia.log
labels:
  type: authelia
---
source: docker
container_name:
 - adguardhome
#container_id:
# - 843ee92d231b
labels:
  type: adguardhome
---
filenames:
 - /var/log/adguard/AdGuardHome.log
labels:
  type: adguardhome
```
{: file="./config/acquis.yaml"}

> You can define a specific path to a text file or even link the complete docker container if your parser support it. For this to work CrowdSec needs a connection to the Docker host usually at `/var/run/docker.sock` or through `Docker socket proxy`.
{: .prompt-info }

## CrowdSec: Whitelisting IPs or domains
Create a file called `whitelist.yaml` and mount it at your container at `/etc/crowdsec/parsers/s02-enrich/whitelists.yaml` for IPs to be whitelisted.
```yml
name: crowdsecurity/whitelists
description: "Whitelist events from private ipv4 addresses"
whitelist:
  reason: "private ipv4/ipv6 ip/ranges/fqdns"
  ip:
    - "127.0.0.1"
    - "::1"
  cidr:
    - "192.168.1.0/24"
    - "172.16.0.0/12"
```
{: file="./config/whitelists.yaml"}

Create a file called `fqdn-whitelist.yaml` and mount it at `/etc/crowdsec/postoverflows/s01-whitelist/fqdn-whitelist.yaml`.
```yml
name: me/fqdn-whitelist
description: lets whitelist our own reverse dns
whitelist:
  reason: dont ban my ISP
  expression:
  # this is the reverse of my ip, you can get it by performing a "host" command on your public IP for example
    - evt.Enriched.reverse_dns endsWith 'yourprovider.de.'
    - evt.Enriched.reverse_dns endsWith '.skynetcloud.org.'
```
{: file="./config/fqdnwhitelist.yaml"}

## CrowdSec: useful commands
See metrics, an overview of parsed logs and decisions:
```bash
docker exec crowdsec cscli metrics
```

See bans (or decisions):
```bash
docker exec crowdsec cscli decisions list
```

Manually install collections:
```bash
docker exec crowdsec cscli collections install crowdsecurity/traefik
```

Update hub:
```bash
docker exec crowdsec cscli hub update
```

Upgrade hub:
```bash
docker exec crowdsec cscli hub upgrade
```

Add bouncer:
```bash
docker exec crowdsec cscli bouncers add bouncer-traefik
```
> Save api key somewhere

Ban IP:
```bash
docker exec crowdsec cscli decisions add --ip 192.168.0.101
```

Unban IP:
```bash
docker exec crowdsec cscli decisions delete --ip 192.168.0.101
```
> Hint: To get shorter commands just add aliases to your bashrc or zshrc.
> Also for collection hub update and upgrade create a script and put it to your systemd.timer or cronjob list. I wrote an article for this with this as an example: [https://docs.skynetcloud.org/posts/schedule-jobs/](https://docs.skynetcloud.org/posts/schedule-jobs/)

## Traefik & Authelia: compose.yml
```yml
services:
  traefik:
    container_name: traefik
    image: traefik:latest
    ports:
      - 80:80
      - 443:443
      - 4243:8080 # Dashboard port
    volumes:
      - ./config:/etc/traefik/
      - traefik-logs:/var/log/traefik
    networks:
      - proxy
    environment:
      DOCKER_HOST: dockersocket
      CF_DNS_API_TOKEN: ${CF_DNS_API_TOKEN}
    restart: unless-stopped
    depends_on:
      - dockersocket
  dockersocket:
    container_name: dockersocket
    image: tecnativa/docker-socket-proxy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - proxy
    environment:
      - BUILD=1
      - COMMIT=1
      - CONFIGS=1
      - CONTAINERS=1
      - DISTRIBUTION=1
      - EXEC=1
      - IMAGES=1
      - INFO=1
      - NETWORKS=1
      - NODES=1
      - PLUGINS=1
      - SERVICES=1
      - SESSSION=1
      - SWARM=0
      - POST=1
    #    privileged: true
    restart: unless-stopped
    
  authelia:
    container_name: authelia
    image: authelia/authelia:latest
    volumes:
      - ./authelia:/config
      - authelia-logs:/var/log/
    labels:
      traefik.enable: true
      traefik.http.routers.authelia.entryPoints: https
      traefik.http.routers.authelia.rule: Host(`${domain}`)
    ports:
      - 9091:9091
    networks:
      - proxy
    restart: unless-stopped
    depends_on:
      - traefik
networks:
  proxy:
    driver: bridge
    external: true
volumes:
  crowdsec-db: null
  crowdsec-config: null
  traefik-logs: null
  authelia-logs: null
```
{: file="traefik/compose.yml"}

## Traefik static configuration file: traefik.yml

```yml
global:
  checkNewVersion: true
  sendAnonymousUsage: false

serversTransport:
  insecureSkipVerify: true

entryPoints:
  # Not used in apps, but redirect everything from HTTP to HTTPS
  http:
    address: :80
    forwardedHeaders:
      trustedIPs: &trustedIps
        - 172.25.0.0/16 # Docker proxy network
        - 172.17.0.0/16 # Docker default bridge
        - 192.168.1.0/24 # Local net
        # Start of Clouflare public IP list for HTTP requests, remove this if you don't use it
        # If you proxy your IPs at Cloudflare you have to uncomment this
#        - 173.245.48.0/20
#        - 103.21.244.0/22
#        - 103.22.200.0/22
#        - 103.31.4.0/22
#        - 141.101.64.0/18
#        - 108.162.192.0/18
#        - 190.93.240.0/20
#        - 188.114.96.0/20
#        - 197.234.240.0/22
#        - 198.41.128.0/17
#        - 162.158.0.0/15
#        - 104.16.0.0/12
#        - 172.64.0.0/13
#        - 131.0.72.0/22
#        - 2400:cb00::/32
#        - 2606:4700::/32
#        - 2803:f800::/32
#        - 2405:b500::/32
#        - 2405:8100::/32
#        - 2a06:98c0::/29
#        - 2c0f:f248::/32
        # End of Cloudlare public IP list
    http:
      middlewares:
        - crowdsecBouncer@file

      redirections:
        entryPoint:
          to: https
          scheme: https

  # HTTPS endpoint, with domain wildcard
  https:
    address: :443
    forwardedHeaders:
      # Reuse list of Cloudflare Trusted IP's above for HTTPS requests
      trustedIPs: *trustedIps
    http:
      tls:
        # Generate a wildcard domain certificate
        certResolver: letsencrypt
        domains:
          - main: skynetcloud.org
            sans:
              - '*.skynetcloud.org'
      middlewares:
        - gzip@file
        - securityHeaders@file
        - crowdsecBouncer@file
  gitea-ssh:
    address: ":2424/tcp"

  wireguard:
    address: ":51822/udp"

providers:
  providersThrottleDuration: 2s
  
  # File provider for connecting things that are outside of docker / defining middleware
  file:
    filename: /etc/traefik/fileConfig.yml
    watch: true

  # Docker provider for connecting all apps that are inside of the docker network
  docker:
    watch: true
    network: proxy # Add Your Docker Network Name Here
    exposedByDefault: false
    endpoint: "tcp://dockersocket:2375" # Uncomment if you are using docker socket proxy
# Enable traefik ui
api:
  dashboard: true
  debug: true
  insecure: true

# Log level INFO|DEBUG|ERROR
log:
  level: INFO
  filePath: "/var/log/traefik/traefik.log"
accessLog:
  filePath: "/var/log/traefik/access.log"

# Use letsencrypt to generate ssl serficiates
certificatesResolvers:
  letsencrypt:
    acme:
      email: admin@skynetcloud.org
      storage: /etc/traefik/acme.json
      dnsChallenge:
        provider: cloudflare
        # Used to make sure the dns challenge is propagated to the rights dns servers
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```
{: file="./config/traefik.yml"}

#### Traefik dynamic configuration file: fileConfig.yml
```yml
http:
  serversTransports:
  # If a service only allows HTTPS connections with custom self-signed certificates you have to allow this
    ignorecert:
      insecureSkipVerify: true

  routers:
    rtr-adguard:
      entrypoints:
        - https
      rule: 'Host(`adguard.skynetcloud.org`)'
      service: svc-adguard

  services:
    svc-adguard:
      loadbalancer:
        servers:
          - url: http://192.168.1.253:80/

  middlewares:

    # Only Allow Local networks
    local-ipwhitelist:
      ipWhiteList:
        sourceRange: 
          - 127.0.0.1/32     # localhost
          - 172.25.0.0/16    # proxy
          - 172.30.0.0/24    # wireguard, adguard
          - 192.168.1.0/24 # LAN Subnet

    # Enable GZip header compression
    gzip:
      compress: true

    # Middleware: Authelia guard SSO for needed websites / services
    auth:
      forwardauth:
        address: http://authelia:9091/api/verify?rd=https://auth.skynetcloud.org/
        trustForwardHeader: true
        authResponseHeaders:
          - Remote-User
          - Remote-Groups
          - Remote-Name
          - Remote-Email

    # Middleware: Authelia basic auth guard
    auth-basic:
      forwardauth:
        address: http://authelia:9091/api/verify?auth=basic
        trustForwardHeader: true
        authResponseHeaders:
          - Remote-User
          - Remote-Groups
          - Remote-Name
          - Remote-Email

    # Security headers for all published websites
    securityHeaders:
      headers:
        customResponseHeaders:
          X-Robots-Tag: "none,noarchive,nosnippet,notranslate,noimageindex"
          server: ""
          X-Forwarded-Proto: "https"
        accessControlAllowOriginList: "*"
        sslProxyHeaders:
          X-Forwarded-Proto: https
        referrerPolicy: "strict-origin-when-cross-origin"
        hostsProxyHeaders:
          - "X-Forwarded-Host"
        customRequestHeaders:
          X-Forwarded-Proto: "https"
        contentTypeNosniff: true
        browserXssFilter: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsSeconds: 63072000
        stsPreload: true

    # Middleware: Crowdsec Traefik Bouncer
    crowdsecBouncer:
      forwardauth:
        address: http://bouncer-traefik:8080/api/v1/forwardAuth
        trustForwardHeader: true

udp:
  routers:
    rtr-wireguard:
      entryPoints:
        - "wireguard"
      service: "svc-wireguard"
  services:
    svc-wireguard:
      loadBalancer:
        servers:
          - address: 192.168.1.4:51822 # Server address to wireguard docker container

# Only use secure ciphers - https://ssl-config.mozilla.org/#server=traefik&version=2.6.0&config=intermediate&guideline=5.6
tls:
  options:
    default:
      minVersion: VersionTLS12
      cipherSuites:
       # Recommended ciphers for TLSv1.2
       - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
       - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
       - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
       # Recommended ciphers for TLSv1.3
       - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
       - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
       - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
```
{: file="./config/fileConfig.yml"}

## Automatically add docker container to Traefik with labels
Add this to your docker compose file:
```yml
networks:
  - proxy
labels:
  traefik.enable: true
  traefik.http.routers.app_name.entryPoints: https
  traefik.http.routers.app_name.rule: Host(`subdomain_name.skynetcloud.org`)
# or
  traefik.http.routers.app_name.rule: (Host(`skynetcloud.org`) && Path(`/app_name/`))

# Optional
  traefik.http.services.app_name.loadbalancer.server.port: 8080
  traefik.http.services.app_name.loadbalancer.server.scheme: http

# Add this to secure your service with Authelia Single-Sign-On
  traefik.http.routers.app_name.middlewares: auth@file

networks:
  proxy:
    driver: bridge
    external: true
```

## Authelia: configuration
As you might understand. I will not show you my configuration here. :-) Go visit the official documentian at [https://www.authelia.com/configuration/prologue/introduction/](https://www.authelia.com/configuration/prologue/introduction/)
