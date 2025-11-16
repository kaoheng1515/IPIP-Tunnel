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
## MikroTik IPIP Tunnel: Local → Hong Kong 
**One tunnel · One routing table · VLAN support · 1:1 fixed HK IP per VLAN or per device**  
Tested & working 100% on RouterOS 7.x – November 2025

### Real IPs used in this example
| Item                        | Value                          |
|-----------------------------|--------------------------------|
| Cambodia WAN IP             | `203.0.113.50` (ether1)        |
| Hong Kong main WAN IP       | `103.123.124.1` (ether1)        |
| Hong Kong extra public IPs  | `103.123.124.2` – `103.123.124.10` |
| Tunnel point-to-point       | `10.0.0.1` (HK) ↔ `10.0.0.2` (KH) |
| Cambodia VLANs (example)    | VLAN10, VLAN20, VLAN30, VLAN88 |

---

## 1. HongKong MikroTik
``routeros
# === Identity ===
# === Identity ===
/system identity set name=HK-MikroTik

# === Add all extra public IPs as /32 on WAN interface ===
/ip address
add address=103.123.124.2/32  interface=ether1 comment="KH dedicated IP"
/add address=103.123.124.3/32  interface=ether1
add address=103.123.124.4/32  interface=ether1
add address=103.123.124.5/32  interface=ether1
add address=103.123.124.6/32  interface=ether1
add address=103.123.124.7/32  interface=ether1
add address=103.123.124.8/32  interface=ether1
add address=103.123.124.9/32  interface=ether1
add address=103.123.124.10/32 interface=ether1

# === IPIP tunnel to Cambodia ===
/interface tunnel
add name=ipip-to-KH local-address=103.123.124.1 remote-address=203.0.113.50 \
    mtu=1476 keepalive=10s,3 comment="Tunnel to Cambodia"

/ip address
add address=10.0.0.1/30 interface=ipip-to-KH network=10.0.0.0

# === Firewall – allow tunnel ===
/ip firewall filter
add chain=input action=accept protocol=ipencap in-interface=ether1 src-address=203.0.113.50 place-before=0
add chain=input action=accept src-address=10.0.0.0/30 in-interface=ipip-to-KH place-before=0

# === Optional (safe) ===
/ip settings set rp-filter=loose










































## Flow – How IPIP Works

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
