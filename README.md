# IPIP-Tunnel
this project setup ipip-tunnel in mikrotik local remote to hongkong

![GitHub](https://img.shields.io/github/license/kaoheng1515/IPIP-Tunnel?style=flat)
![GitHub last commit](https://img.shields.io/github/last-commit/kaoheng1515/IPIP-Tunnel?style=flat)
![ViewCount](https://views.whatilearened.today/views/github/kaoheng1515/lIPIP-Tunnel.svg?cache=remove)

## What is the IP-IP Tunnel Protocol?
- **Feature**          **Details**
- **Full Name**        IP-in-IP encapsulation (RFC 2003)
- **OSI Layer**        Layer 3 (Network layer) → IP packet inside another IP packet
- **Type**             "Stateless connectionless, very lightweight tunneling protocol"
- **Encryption**       None (plain text – same as normal Internet traffic)
- **Typical Use Case** "Move your public IP from one location to another (e.g., your servers appear in Hong Kong instead of your home country)"MikroTik name,/interface ipip

# Flow IPIP Working
 Local Router (203.0.113.10)                     Hong Kong VPS (103.123.456.10)
┌─────────────────────┐    IP-IP packet    ┌─────────────────────┐
│ Your server         │ ─────────────────► │ Hong Kong router    │
│ 192.168.88.11       │  Original packet:  │ decapsulates →      │
│ wants to visit      │  Src: 192.168.88.11│ forwards with       │
│ google.com          │  Dst: 8.8.8.8      │ Src: 103.123.456.x  │
└─────────────────────┘                    └─────────────────────┘
          │                                         ▲
          └─────► Encapsulates into new IP header:
                 Outer Src: 203.0.113.10
                 Outer Dst: 103.123.456.10
                 Protocol = 4 (IP-IP)
                 Inner packet stays unchanged
