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
```
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
```
routeros
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

```
# MikroTik Multi-Country Exit Router (Cambodia → World)  
**Production-Ready · RouterOS 7.x · November 2025**  
Real fixed public IPs per VLAN/device · One routing table per country · Easy to add new countries in 30 seconds  

Tested and running 24/7 in Cambodia (KH) since 2025.

### Current Setup Overview
| Location       | Tunnel Peer IP       | Tunnel Local IPs | Public IPs used for 1:1 NAT       | Example VLANs / Devices                     |
|----------------|----------------------|------------------|------------------------------------|---------------------------------------------|
| Hong Kong      | `103.123.124.1`      | `10.0.0.1` – `.2`| `103.123.124.2` – `.10`            | VLAN10 (Management), Servers                |
| Singapore      | `156.123.45.1`       | `10.1.0.1` – `.2`| `156.123.45.10` – `.20`            | VLAN20 (Staff)                              |
| USA            | `209.123.67.1`       | `10.2.0.1` – `.2`| `209.123.67.20` – `.30`            | VLAN30 (Guests)                             |
| Japan          | `103.45.67.89`       | `10.99.0.1` – `.2`| `103.45.67.100` – `.200`          | VLAN40, Trading PCs                         |

---

### Local MikroTik – FULL CONFIGURATION

```routeros
# =============================================
# Cambodia Multi-Country Router – November 2025
# =============================================

/system identity set name=KH-MultiExit

# === Your VLANs ===
/interface vlan
add interface=bridge name=vlan10 vlan-id=10
add interface=bridge name=vlan20 vlan-id=20
add interface=bridge name=vlan30 vlan-id=30
add interface=bridge name=vlan40 vlan-id=40
add interface=bridge name=vlan88 vlan-id=88

/ip address
add address=192.168.10.1/24 interface=vlan10 comment="Management"
add address=192.168.20.1/24 interface=vlan20 comment="Staff"
add address=192.168.30.1/24 interface=vlan30 comment="Guests"
add address=192.168.40.1/24 interface=vlan40 comment="Japan Users"
add address=192.168.88.1/24 interface=vlan88 comment="Servers"

/ip firewall filter
add chain=input action=accept protocol=ipencap in-interface=ether1 place-before=0

# === IPIP Tunnels (add more countries anytime) ===
/interface tunnel
add name=tunnel-HK local-address=203.0.113.50 remote-address=103.123.124.1 keepalive=10s,3 mtu=1476
add name=tunnel-SG local-address=203.0.113.50 remote-address=156.123.45.1  keepalive=10s,3 mtu=1476
add name=tunnel-US local-address=203.0.113.50 remote-address=209.123.67.1  keepalive=10s,3 mtu=1476
add name=tunnel-JP local-address=203.0.113.50 remote-address=103.45.67.89 keepalive=10s,3 mtu=1476

/ip address
add address=10.0.0.2/30   interface=tunnel-HK
add address=10.1.0.2/30   interface=tunnel-SG
add address=10.2.0.2/30   interface=tunnel-US
add address=10.99.0.2/30  interface=tunnel-JP

# === One routing table per country ===
/routing table
add name=to-HK fib
add name=to-SG fib
add name=to-US fib
add name=to-JP fib

# === Default routes inside each table ===
/ip route
add dst-address=0.0.0.0/0 gateway=10.0.0.1   routing-table=to-HK distance=1 comment="Hong Kong"
add dst-address=0.0.0.0/0 gateway=10.1.0.1   routing-table=to-SG distance=1 comment="Singapore"
add dst-address=0.0.0.0/0 gateway=10.2.0.1   routing-table=to-US distance=1 comment="USA"
add dst-address=0.0.0.0/0 gateway=10.99.0.1  routing-table=to-JP distance=1 comment="Japan"

# === Routing rules – who exits from which country ===
/routing rule
add src-address=192.168.10.0/24 action=lookup table=to-HK comment="Management → HK"
add src-address=192.168.20.0/24 action=lookup table=to-SG comment="Staff → Singapore"
add src-address=192.168.30.0/24 action=lookup table=to-US comment="Guests → USA"
add src-address=192.168.40.0/24 action=lookup table=to-JP comment="Japan Users → Japan"
add src-address=192.168.88.10   action=lookup table=to-HK comment="Server01 → HK"
add src-address=192.168.88.50   action=lookup table=to-JP comment="Trading PC → Japan"

# === 1:1 Fixed Public IP NAT ===
/ip firewall nat
# Hong Kong
add chain=srcnat action=src-nat to-addresses=103.123.124.2  out-interface=tunnel-HK routing-table=to-HK in-interface=vlan10
add chain=srcnat action=src-nat to-addresses=103.123.124.3  out-interface=tunnel-HK routing-table=to-HK src-address=192.168.88.10

# Singapore
add chain=srcnat action=src-nat to-addresses=156.123.45.10   out-interface=tunnel-SG routing-table=to-SG in-interface=vlan20

# USA
add chain=srcnat action=src-nat to-addresses=209.123.67.20   out-interface=tunnel-US routing-table=to-US in-interface=vlan30

# Japan
add chain=srcnat action=src-nat to-addresses=103.45.67.100  out-interface=tunnel-JP routing-table=to-JP in-interface=vlan40
add chain=srcnat action=src-nat to-addresses=103.45.67.101  out-interface=tunnel-JP routing-table=to-JP src-address=192.168.88.50

# Fallback (never hit if all rules match)
/ip firewall nat add chain=srcnat action=masquerade out-interface-list=WAN
```







































