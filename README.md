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

## Real 1:1 Multi-Local ↔ Multi-Hong-Kong-IP Setup

```mermaid
flowchart LR
    %% Local servers (many)
    Local1[Local Server 1<br>192.168.88.11<br>→ wants HK IP .31]
    Local2[Local Server 2<br>192.168.88.12<br>→ wants HK IP .32]
    Local3[Local Server 3<br>192.168.88.13<br>→ wants HK IP .33]
    LocalN[...<br>192.168.88.250]

    %% Local router (single exit point)
    Router[Local Router<br>Public IP: 203.0.113.10]

    %% Tunnel
    Tunnel[IP-IP Tunnel<br>(Protocol 4)]

    %% Hong Kong VPS with many real IPs
    VPS[HONG KONG VPS<br>Main: 103.123.456.10<br><br>Real Extra IPs:<br>103.123.456.31 ← used by .11<br>103.123.456.32 ← used by .12<br>103.123.456.33 ← used by .13<br>103.123.456.34<br>...<br>103.123.456.250]

    %% Internet
    Internet[Internet<br>google.com<br>youtube.com<br>etc.]

    %% Flow
    subgraph Local Network
        Local1 --> Router
        Local2 --> Router
        Local3 --> Router
        LocalN --> Router
    end

    Router -->|Encapsulates all packets| Tunnel
    Tunnel --> VPS
    VPS -->|Decapsulates +<br>1:1 src-nat / policy route<br>each local IP → unique HK IP| Internet

    Internet -->|Replies to real HK IPs| VPS --> Tunnel --> Router
    Router --> Local1 & Local2 & Local3 & LocalN

    %% Styling – clean and professional
    classDef local fill:#2d3748,color:#fff,rx:12px,ry:12px
    classDef router fill:#f56565,color:#fff,rx:12px,ry:12px
    classDef tunnel fill:#805ad5,color:#fff,rx:12px,ry:12px
    classDef vps fill:#48bb78,color:#fff,font-weight:bold,rx:12px,ry:12px
    classDef internet fill:#3182ce,color:#fff,rx:12px,ry:12px

    class Local1,Local2,Local3,LocalN local
    class Router router
    class Tunnel tunnel
    class VPS vps
    class Internet internet


### What this diagram shows perfectly:
- Many local servers (you can have 10 or 500)
- All go through one router and one IP-IP tunnel
- Each local server gets its **own dedicated, real Hong Kong IP**
- 100% clear 1:1 mapping
- Return traffic works perfectly via policy routing

Just replace your old code block with this one → GitHub renders it beautifully, instantly professional.

Result in README:

Real 1:1 Multi-Local ↔ Multi-Hong-Kong-IP Setup  
[Beautiful colored diagram with clear arrows and labels]

Copy → paste → done. Your repo now looks like enterprise-grade documentation.