# Pi-hole — Ad Blocking + Local DNS

Pi-hole serves as the DNS server for all VPN clients. It blocks ads and trackers, resolves internal `.local` hostnames to Traefik, and forwards all external queries to Unbound.

## Install

```bash
git clone --depth 1 https://github.com/pi-hole/pi-hole.git Pi-hole
cd "Pi-hole/automated install/"
sudo bash basic-install.sh
```

During setup, set upstream DNS to `127.0.0.1#5335` (Unbound).

## Point Pi-hole upstream to Unbound

In the Pi-hole admin UI:
```text
Settings → DNS → Custom upstream DNS server
Upstream 1: 127.0.0.1#5335
Uncheck all other upstream providers
```

## Local DNS records — internal hostnames

In Pi-hole admin UI → **Local DNS → DNS Records**, add:

```text
portainer.local  →  10.0.1.23
truenas.local    →  10.0.1.23
bitwarden.local  →  10.0.1.23
```

All internal hostnames resolve to Traefik's IP. Traefik then routes to the correct service based on the hostname.

## CNAME records (optional — cleaner naming)

```text
www.portainer.local  →  portainer.local
```

## Set Pi-hole as DNS on WireGuard clients

In the WireGuard client config, the `DNS = 10.0.1.41` line routes all DNS through Pi-hole automatically when the VPN is connected.
