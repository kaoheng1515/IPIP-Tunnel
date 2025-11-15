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
## Flow – How IP-IP Works with Multi-IP Hong Kong VPS

```mermaid
flowchart LR
    %% Nodes – must be quoted when containing ( ) or \n
    A["Local Server<br>192.168.88.11<br>wants google.com"]
    B["Local Router<br>203.0.113.10"]
    C["IP-IP Tunnel<br>(Protocol 4)"]
    D["HONG KONG VPS<br>Main: 103.123.456.10<br><br>Extra IPs on lo:<br>103.123.456.11<br>103.123.456.12<br>103.123.456.13<br>103.123.456.14<br>103.123.456.15 ← current<br>103.123.456.16<br>… up to .250"]
    E["Internet<br>google.com<br>youtube.com<br>etc."]

    %% Connections
    A --> B
    B -->|"1. Encapsulates packet"| C
    C --> D
    D -->|"2. Decapsulates<br>3. Forwards using real HK IP"| E

    %% Return path
    E -->|"Reply → 103.123.456.15"| D
    D -->|"4. Policy route → tunnel"| C
    C --> B --> A

    %% Styling
    classDef local    fill:#1e293b, color:#fff
    classDef router   fill:#dc2626, color:#fff
    classDef tunnel   fill:#7c3aed, color:#fff
    classDef vps      fill:#16a34a, color:#fff, font-weight:bold
    classDef internet fill:#0ea5e9, color:#fff

    class A local
    class B router
    class C tunnel
    class D vps
    class E internet
    ## Quick Notes – What You Must Do on Each Router

### 1. Your Local MikroTik Router (203.0.113.10)

| What to configure | Exact commands (copy-paste) |
|-------------------|-----------------------------|
| Create IP-IP tunnel | `/interface ipip add name=ipip-hk remote-address=103.123.456.10 keepalive=10s` |
| Tunnel addresses | `/ip address add address=10.0.0.2/30 interface=ipip-hk` |
| Default route via tunnel | `/ip route add gateway=10.0.0.1 distance=1` |
| Current HK IP (change anytime) | `/ip firewall nat add chain=srcnat action=src-nat to-addresses=103.123.456.15 out-interface=ipip-hk comment="HK IP"` |
| To switch IP instantly | `/ip firewall nat set [find comment="HK IP"] to-addresses=103.123.456.89` |

That’s all on the local MikroTik – no scripts needed.

### 2. Hong Kong VPS (103.123.456.10) – Linux one-time only

```bash
# Enable forwarding
sysctl -w net.ipv4.ip_forward=1

# Create tunnel (accepts from any of your IPs)
ip tunnel add tun0 mode ipip remote any local 103.123.456.10
ip addr add 10.0.0.1/30 dev tun0
ip link set tun0 up

# Add all your extra HK IPs to loopback
for i in {11..250}; do
  ip addr add 103.123.456.$i/32 dev lo
done

# Return routing (so replies go back into tunnel)
ip route add 192.168.88.0/24 dev tun0 scope link     # ← your local LAN here
echo "200 hk" >> /etc/iproute2/rt_tables
for i in {11..250}; do
  ip rule add from 103.123.456.$i lookup hk
  ip route add default via 10.0.0.2 dev tun0 table hk
done