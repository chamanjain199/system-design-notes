# 🌐 Networking Fundamentals — From NICs to Cloud

> A complete beginner-friendly reference covering physical hardware, IP addressing, routing, and cloud networking.

---

## Table of Contents
- [Cloud Instances & Physical Machines](#cloud-instances--physical-machines)
- [What is a NIC?](#what-is-a-nic)
- [MAC Address](#mac-address)
- [Wired vs Wireless NICs](#wired-vs-wireless-nics)
- [Why Routers Have Multiple NICs](#why-routers-have-multiple-nics)
- [IP Addresses & NICs](#ip-addresses--nics)
- [Domain Registration & Public IP](#domain-registration--public-ip)
- [Dynamic vs Static IP](#dynamic-vs-static-ip)

---

## Cloud Instances & Physical Machines

Every cloud instance (like AWS EC2) ultimately runs on a **real physical server** in a data center.

### Key concepts

- One physical machine hosts **many virtual instances** via a **hypervisor** (e.g., AWS Nitro, KVM, VMware)
- The hypervisor slices CPU, RAM, and network across VMs
- Instance types determine how dedicated the hardware is:

| Instance Type | Physical Mapping |
|---|---|
| `t3.micro` (shared) | Many VMs share one physical host |
| Dedicated instance | No other customers on same host |
| `i3.metal` (bare metal) | 1:1 mapping — direct hardware access |

> **Note:** Storage (EBS) is usually on **separate hardware**, accessed over a fast internal network.

### AWS Nitro
AWS uses a custom hardware card called the **Nitro Card** that offloads all networking and storage I/O off the main CPU — freeing it entirely for your workloads.

---

## What is a NIC?

**NIC = Network Interface Card** — the hardware that connects a device to a network.

- Every NIC can **transmit (TX) and receive (RX)** simultaneously — called **full duplex**
- In an Ethernet cable, separate wire pairs handle TX and RX

```
Ethernet Cable (8 wires = 4 pairs)
  ├── Pair 1 → TX (sending)
  ├── Pair 2 → RX (receiving)
  └── Pair 3,4 → used in Gigabit+
```

### NIC Types

| NIC Type | Medium | Protocol | Typical Use |
|---|---|---|---|
| Ethernet NIC | Copper wire | 802.3 | PCs, servers, data centers |
| Fiber NIC (SFP) | Fiber optic | 802.3 | Data centers, long distance |
| WiFi NIC | Radio waves | 802.11 a/b/g/n/ac/ax | Laptops, phones |
| Cellular modem | Radio waves | LTE / 5G | Mobile devices |

> One NIC **cannot** handle both wired and wireless — they use different hardware, connectors, and signal types.

### Virtual NICs (vNIC)
In cloud/virtual environments, each VM gets a **software-simulated NIC** (vNIC). The hypervisor acts as a virtual switch, multiplexing all VM traffic through the one or two physical NICs on the host.

---

## MAC Address

**MAC = Media Access Control** — a unique hardware identifier burned into each NIC at manufacturing time.

```
Example:  A4:C3:F0:85:2D:11
          ───────────  ───────────
          OUI           Device ID
     (Manufacturer)   (Unique per device)
          (3 bytes)       (3 bytes)
```

### MAC vs IP

| | MAC Address | IP Address |
|---|---|---|
| Assigned by | Manufacturer (hardware) | Network admin / DHCP (software) |
| Scope | Local network only (Layer 2) | Global / routable (Layer 3) |
| Changes? | Permanent (but can be spoofed) | Changes when you switch networks |
| Purpose | Identify device on local segment | Identify device across internet |

> Think of **MAC** as your name (who you are) and **IP** as your mailing address (where you are).

### Key Rules
- Every NIC has its own MAC address
- A device with 2 NICs has **2 MAC addresses**
- MAC addresses are **stripped at every router hop** — they only travel within a local network segment
- Virtual NICs get **software-assigned MAC addresses** managed by the cloud provider

---

## Wired vs Wireless NICs

They are **fundamentally different hardware** — a wired NIC has no antenna, a wireless NIC has no port.

### Why routers separate wired and wireless networks

Physically different signals, but the router distinguishes them logically using **VLANs**:

| Reason | Explanation |
|---|---|
| **Security** | Wired = physically harder to access = more trusted |
| **Performance** | Wired is stable; WiFi is variable and shared |
| **IoT isolation** | Smart devices kept away from main network |
| **Guest networks** | Visitors can't reach your internal devices |

### VLANs (Virtual LANs)
```
Physical: all traffic flows through same router chips
Logical:
  VLAN 10 → Wired devices    (192.168.1.x)
  VLAN 20 → WiFi devices     (192.168.2.x)
  VLAN 30 → Guest WiFi       (192.168.3.x)
  VLAN 40 → IoT devices      (192.168.4.x)
```

> Wired and wireless **don't have to** be separate — many home routers bridge them into one network.

---

## Why Routers Have Multiple NICs

A router's job is to **connect multiple networks** and forward traffic between them.

```
Network A          ROUTER          Network B
192.168.1.x  ←──[NIC1 | NIC2]──→  10.0.0.x
```

With only 1 NIC, a router could only join one network — it couldn't route between networks.

### Home Router Example

```
Internet ───── WAN NIC  → Public IP (103.x.x.x)
Wired PCs ──── LAN NIC  → 192.168.1.1
WiFi devices ─ WiFi NIC → 192.168.1.1
```

### How a Packet Travels
1. Your PC sends packet to router's LAN NIC
2. Router checks routing table — destination is outside LAN
3. Router forwards via WAN NIC to internet
4. Reply comes back through WAN NIC → forwarded to your PC

### Two Physical NICs = Redundancy
The typical reason for 2 NICs on a server is:
- **Redundancy** — if one fails, traffic failover to the other
- **Bonding** — combine both for higher throughput (e.g., 2 × 25 Gbps)

---

## IP Addresses & NICs

**Each NIC gets its own IP address.**

```
Laptop
  ├── Ethernet NIC → 192.168.1.10
  └── WiFi NIC     → 192.168.1.15
```

### One NIC, Multiple IPs (IP Aliasing)
A single NIC can have multiple IPs assigned:
```
Single NIC
  ├── Primary:  192.168.1.10
  ├── Alias 1:  192.168.1.11
  └── Alias 2:  192.168.1.12
```
Used for: hosting multiple websites, failover scenarios, testing.

### Special IPs

| IP | Name | Purpose |
|---|---|---|
| `127.0.0.1` | Loopback | Talk to yourself — never leaves machine |
| `169.254.x.x` | APIPA | Auto-assigned when no DHCP found |

### How to Check All IPs on Router

| Method | What you see |
|---|---|
| `ipconfig` / `ip route` | Router's LAN IP only |
| Router admin panel (`192.168.1.1`) | All IPs — WAN + LAN + WiFi |
| `curl ifconfig.me` | Router's public WAN IP only |
| SSH into router → `ip addr` | All IPs with full detail |
| `nmap -sn 192.168.1.0/24` | All device IPs on local network |

---

## Domain Registration & Public IP

When registering a domain, you use your **Public WAN IP** — not your device's private IPs.

### Private IPs are invisible to the internet

```
Internet
    │
    │  ← only PUBLIC IP visible here
    │
Router WAN: 103.45.67.89  ← what internet sees
    │
    ├── PC NIC 1: 192.168.1.10  ← INVISIBLE to internet
    └── PC NIC 2: 192.168.1.15  ← INVISIBLE to internet
```

### Private IP Ranges (never routable on internet)
```
10.0.0.0    – 10.255.255.255
172.16.0.0  – 172.31.255.255
192.168.0.0 – 192.168.255.255
```

### DNS A Record
```
mysite.com  →  A record  →  103.45.67.89 (Public IP)
```

### Full request journey
```
User types mysite.com
    → DNS resolves to Public IP: 103.45.67.89
    → Reaches Router's WAN NIC
    → Port Forwarding: forward port 80 → 192.168.1.10
    → Reaches your PC's NIC
    → Web server responds
```

---

## Dynamic vs Static IP

### Dynamic IP
ISP assigns IPs from a shared pool using DHCP leases. Your IP can change when:
- Router restarts
- DHCP lease expires
- Power cut occurs

**Why ISPs use dynamic IPs:** IPv4 shortage (~4.3 billion addresses, ~5.4 billion users). Not everyone is online simultaneously, so IPs are recycled.

### Problem for hosting
```
Monday:    mysite.com → 103.45.67.89  ✅
Wednesday: router restarts → new IP: 103.45.67.91
           mysite.com → still 103.45.67.89  ❌ broken!
```

### Solutions

| Solution | How it works | Best for |
|---|---|---|
| **DDNS** | Auto-updates DNS when IP changes (No-IP, DuckDNS) | Home hosting |
| **Static IP from ISP** | Pay extra for permanent IP | Small businesses |
| **Cloud server** | EC2/VPS always gets static public IP | Production websites |

### IPv6 — Long term solution
IPv6 provides **340 undecillion addresses** — enough for every device to have a permanent public IP, eliminating the need for dynamic IPs. Adoption is still in progress worldwide.

---

## Quick Reference

```
Physical NIC → has MAC address → gets IP address
     ↓
Hypervisor creates virtual NICs for each VM
     ↓
Each vNIC gets its own MAC + private IP
     ↓
Router has multiple NICs to connect multiple networks
     ↓
Router's WAN NIC has the public IP used in DNS
     ↓
Domain points to public IP → router port forwards to private IP
```

---

*Notes compiled from networking fundamentals study session.*
