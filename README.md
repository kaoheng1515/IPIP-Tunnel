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
## MikroTik → Real Hong Kong Multi-IP Setup (2025 Production Version)

```mermaid
flowchart LR
    A["Your Local Devices<br>192.168.88.0/24<br>Phones, PCs, VMs, Docker…"]
    B["Your MikroTik Router<br>WAN IP: 203.0.113.10"]
    C["IP-IP Tunnel<br>/interface ipip"]
    D["HONG KONG VPS<br>Main IP: 103.123.456.10<br><br>200+ Real IPs on lo:<br>103.123.456.11<br>103.123.456.12 ← current<br>103.123.456.13<br>…<br>103.123.456.250"]
    E["Internet<br>Google · YouTube · TikTok<br>Apple · OpenAI · etc."]

    A --> B
    B -->|"1. All traffic → ipip tunnel"| C
    C --> D
    D -->|"2. Decapsulates<br>3. src-nat → real HK IP"| E
    E -->|"Reply → 103.123.456.12"| D
    D -->|"4. Policy routing → back into tunnel"| C --> B --> A

    classDef local    fill:#1e293b, color:#fff
    classDef mikrotik fill:#dc2626, color:#fff, stroke:#991b1b, stroke-width:3px
    classDef tunnel   fill:#7c3aed, color:#fff
    classDef vps      fill:#16a34a, color:#fff, font-weight:bold, stroke:#15803d
    classDef inet     fill:#0ea5e9, color:#fff

    class A local
    class B mikrotik
    class C tunnel
    class D vps
    class E inet