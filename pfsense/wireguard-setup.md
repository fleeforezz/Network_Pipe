# WireGuard on pfSense (GUI setup)

If WireGuard is running directly on pfSense rather than a separate Linux server, use the built-in GUI instead of a `wg0.conf` file. The steps below mirror the pfSense web interface exactly.

## Step 1 — Install the WireGuard package

```text
System → Package Manager → Available Packages
Search: wireguard → Install
```

> pfSense 2.6+ has WireGuard built in — skip this step if the VPN menu already shows a WireGuard entry.

## Step 2 — Create the tunnel

```text
VPN → WireGuard → Tunnels → Add Tunnel
```

| Field | Value |
|---|---|
| Description | `HomeLab VPN` |
| Listen Port | `51820` |
| Interface addresses | `10.8.0.1/24` |
| Keys | Click **Generate** — copy the public key for client configs |

> The VPN subnet `10.8.0.0/24` must not overlap your LAN `10.0.1.0/24`.

## Step 3 — Add a peer (one per client device)

```text
VPN → WireGuard → Peers → Add Peer
```

| Field | Value |
|---|---|
| Tunnel | `tun_wg0 (HomeLab VPN)` |
| Description | `My Laptop` |
| Public Key | `<CLIENT_PUBLIC_KEY>` — generated on the client (see step 5) |
| Allowed IPs | `10.8.0.2/32` — one unique IP per device |
| Keepalive | `25` — prevents NAT timeout on mobile connections |

> Add one peer per device. Laptop = `.2`, phone = `.3`, work PC = `.4`, etc. Never reuse key pairs.

## Step 4 — Assign the WireGuard interface

```text
Interfaces → Assignments → Add tun_wg0
```

| Field | Value |
|---|---|
| Enable | ✓ checked |
| Description | `WIREGUARD` |
| IPv4 config type | `Static IPv4` |
| IPv4 address | `10.8.0.1 / 24` |

Save and apply changes.

## Step 5 — Firewall rules

**WAN tab — allow VPN traffic in from the internet:**

```text
Firewall → Rules → WAN → Add
Action    : Pass
Protocol  : UDP
Dest.     : WAN address, port 51820
```

**WIREGUARD interface tab — allow VPN clients to reach LAN:**

```text
Firewall → Rules → WIREGUARD → Add
```

| Rule | Action | Source | Destination | Protocol |
|---|---|---|---|---|
| Allow VPN → LAN | Pass | `10.8.0.0/24` | `10.0.1.0/24` | TCP/UDP |
| Allow VPN → Pi-hole DNS | Pass | `10.8.0.0/24` | `10.0.1.41 :53` | UDP |
| Block peer-to-peer | Block | `10.8.0.0/24` | `10.8.0.0/24` | any |

> Rules are evaluated top-to-bottom — first match wins. Place the block rule last.

## Step 6 — Configure the client device

Generate a key pair on the client:

```bash
wg genkey | tee client_private.key | wg pubkey > client_public.key
cat client_public.key   # paste this into pfSense peer → Public Key field
```

Client config file:

```ini
[Interface]
Address    = 10.8.0.2/24
PrivateKey = <CLIENT_PRIVATE_KEY>
DNS        = 10.0.1.41        # Pi-hole — local DNS resolution over VPN

[Peer]
PublicKey  = <PFSENSE_PUBLIC_KEY>   # copied from tunnel step 2
Endpoint   = <YOUR_WAN_IP>:51820
AllowedIPs = 10.0.1.0/24           # split tunnel — LAN traffic only
             # use 0.0.0.0/0 for full tunnel (all traffic via VPN)
PersistentKeepalive = 25
```

## Step 7 — Verify the connection

```bash
# Check tunnel handshake in pfSense:
# VPN → WireGuard → Status — peer should show latest handshake + transfer stats

# From the connected client, test each layer in order:
ping 10.0.1.41                        # 1. reach Pi-hole by IP
nslookup portainer.local           # 2. local DNS resolves to 10.0.1.23
nslookup google.com                   # 3. external DNS resolves via Unbound
# Then open https://portainer.local in browser
```
