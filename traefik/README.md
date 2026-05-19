# Traefik — Reverse Proxy

Traefik terminates HTTPS for all internal services. Because Pi-hole resolves all `*.local` hostnames to Traefik's IP, no service needs to be directly accessible by IP or exposed to the internet.

## `docker-compose.yml`

```yaml
version: "3.8"

services:
  traefik:
    image: traefik:v3.0
    container_name: traefik
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
      - ./certs:/certs
    networks:
      - proxy

networks:
  proxy:
    external: true
```

## `traefik.yml`

```yaml
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: ":443"

providers:
  docker:
    exposedByDefault: false

tls:
  certificates:
    - certFile: /certs/homelab.crt
      keyFile: /certs/homelab.key
```

## Adding a service behind Traefik (example: Portainer)

```yaml
services:
  portainer:
    image: portainer/portainer-ce
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`portainer.local`)"
      - "traefik.http.routers.portainer.entrypoints=websecure"
      - "traefik.http.routers.portainer.tls=true"
      - "traefik.http.services.portainer.loadbalancer.server.port=9443"
```
