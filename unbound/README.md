# Unbound — Recursive DNS Resolver

Unbound queries authoritative DNS servers directly, removing dependency on upstream resolvers (Google 8.8.8.8, Cloudflare 1.1.1.1, etc.) and preventing DNS query leakage.

## Install

```bash
sudo apt install unbound -y
```

## Config `/etc/unbound/unbound.conf.d/homelab.conf`

```yaml
server:
  verbosity: 1
  interface: 127.0.0.1
  port: 5335
  do-ip4: yes
  do-udp: yes
  do-tcp: yes

  # DNSSEC validation
  harden-dnssec-stripped: yes
  harden-glue: yes

  # Privacy hardening
  hide-identity: yes
  hide-version: yes

  # Performance
  prefetch: yes
  num-threads: 1

  # Root hints — download periodically
  root-hints: "/var/lib/unbound/root.hints"
```

## Download root hints

```bash
wget -O /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
sudo systemctl restart unbound
```

## Test Unbound is resolving correctly

```bash
dig google.com @127.0.0.1 -p 5335
# Should return an ANSWER SECTION with IP addresses
```
