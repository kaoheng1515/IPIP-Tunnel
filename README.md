# IPIP-Tunnel
this project setup ipip-tunnel in mikrotik local remote to hongkong

![GitHub](https://img.shields.io/github/license/kaoheng1515/IPIP-Tunnel?style=flat)
![GitHub last commit](https://img.shields.io/github/last-commit/kaoheng1515/IPIP-Tunnel?style=flat)
![ViewCount](https://views.whatilearened.today/views/github/kaoheng1515/lIPIP-Tunnel.svg?cache=remove)

# Hong Kong Multi-IP Outbound via IP-IP Tunnel

## What is the IP-IP Tunnel Protocol?

- **Full Name**: IP-in-IP Encapsulation (RFC 2003)
- **OSI Layer**: Layer 3 (Network layer) → IP packet inside another IP packet
- **Type**: Stateless, ultra-lightweight tunneling protocol
- **Encryption**: None (plain text — same as normal Internet traffic)
- **Typical Use Case**: "Move" your public IP from one location to another  
  → Your servers physically in Europe/USA but appear 100% in Hong Kong

> MikroTik name: `/interface ipip`  
> Linux name: `ip tunnel add mode ipip`

## Flow – How IPIP Works

```mermaid
flowchart LR
    A[Local Server<br>192.168.88.11<br>wants google.com] 
    --> B[Local Router<br>203.0.113.10]
    B -->|Encapsulates| C[IP-IP Tunnel]
    C --> D[Hong Kong VPS<br>103.123.456.10]
    D -->|Decapsulates & forwards<br>with Src: 103.123.456.x| E[Internet<br>google.com]

    style A fill:#2d3748,color:#fff
    style B fill:#f56565,color:#fff
    style D fill:#48bb78,color:#fff
    style E fill:#3182ce,color:#fff
