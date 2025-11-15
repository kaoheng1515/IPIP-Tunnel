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

## Real 1:1 Multi-Local to Multi-Hong-Kong-IP Setup

```mermaid
flowchart LR
    %% Nodes – wrapped safely in quotes
    A["Local Server 1<br>192.168.88.11 → HK IP .31"]
    B["Local Server 2<br>192.168.88.12 → HK IP .32"]
    C["Local Server 3<br>192.168.88.13 → HK IP .33"]
    Z["...<br>192.168.88.250"]

    R["Local Router<br>203.0.113.10"]
    T["IP-IP Tunnel<br>(Protocol 4)"]
    V["HONG KONG VPS<br>Main IP: 103.123.456.10<br><br>Real extra IPs:<br>103.123.456.31 ← used by .11<br>103.123.456.32 ← used by .12<br>103.123.456.33 ← used by .13<br>...<br>103.123.456.250"]
    I["Internet<br>google.com<br>etc."]

    %% Connections
    subgraph "Local Network"
        A --> R
        B --> R
        C --> R
        Z --> R
    end

    R -->|"Encapsulates"| T --> V
    V -->|"Decapsulates +<br>1:1 mapping"| I
    I -->|"Replies to real HK IPs"| V --> T --> R

    %% Styling
    classDef local fill:#1e293b, color:#fff, stroke:#475569
    classDef router fill:#dc2626, color:#fff, stroke:#991b1b
    classDef tunnel fill:#7c3aed, color:#fff, stroke:#6d28d9
    classDef vps fill:#16a34a, color:#fff, stroke:#15803d, font-weight:bold
    classDef inet fill:#0ea5e9, color:#fff, stroke:#0369a1

    class A,B,C,Z local
    class R router
    class T tunnel
    class V vps
    class I inet


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