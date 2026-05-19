# pfSense Firewall Rules (standalone reference)

## NAT — Port forward WireGuard

```text
Interface  : WAN
Protocol   : UDP
Dest. port : 51820
Redirect to: <WireGuard server LAN IP>:51820
```

## LAN rules — allow VPN clients to reach services

```text
Action     : Pass
Interface  : LAN
Source     : 10.8.0.0/24   (WireGuard subnet)
Destination: 10.0.1.0/24   (HomeLab LAN)
Protocol   : TCP/UDP
```

## Block VPN clients from accessing each other (optional hardening)

```text
Action     : Block
Interface  : LAN
Source     : 10.8.0.0/24
Destination: 10.8.0.0/24
```
