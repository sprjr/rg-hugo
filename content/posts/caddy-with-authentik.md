---
date: 2024-10-10
# description: ""
# image: ""
lastmod: 2024-10-10
showTableOfContents: false
# tags: ["",]
title: "Caddy With Authentik"
type: "post"
draft: "true"
---

## Configure Caddy to Properly Work with Containers on the Same Host

This post assumes that you have a working instance of Authentik set up, and they're you're familiar with docker/docker-compose, although I'll use docker-compose throughout this content. At the time of writing this, I have both Caddy and Authentik set up as containers.

### Caddy

To get started, here's my compose file for Caddy. Full docs can be found at https://caddyserver.com/.

```
services:
  caddy:
    image: caddy:latest
    container_name: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./site:/srv
      - caddy_data:/data
      - caddy_config:/config
    network_mode: host
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  caddy_data:
  caddy_config:
```

It's worth noting that I set up two volumes as docker volumes, not mounted on the host filesystem itself (`caddy_data` and `caddy_config`). I have never needed to change this. I've also noticed that ./site has never populated any data through my usage, nor have I ever need to put any in. The most edited file for me as been the Caddyfile. If you're not familiar with Caddy and how to handle reverse proxying or similar tasks, it's primarily accomplished by configuring the Caddyfile.

A simple example configuration of the Caddyfile might look like this:

```
https://subdomain.example.com:443 {
    reverse_proxy hostname:port
}
```
All this does is direct HTTPS traffic to whatever target you are running the service on. Unfortunately, if you're running on containers, stuff like routing and dns and hostnames and IP address can get quite confusing quite quickly.
As can be seen in the compose file I provided above, the lines below are present. This can, in certain configurations, entirely remove the issue of having to set up docker networks or handle docker's implementation of DNS. I'm not the most knowledgeable about networking, but in any case it fixed my issues.

```
    extra_hosts:
      - "host.docker.internal:host-gateway"
```
