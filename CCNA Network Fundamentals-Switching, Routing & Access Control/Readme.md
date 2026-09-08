# Cisco Networking Concepts - Switching & VLAN

> Concept notes based on the CCNA (Routing and Switching Essentials) curriculum.
> Order: Switching Fundamentals > VLAN > Dynamic Routing > ACL > Firewall
> Each concept is cross linked so it maps 1:1 to a Packet Tracer lab.


---

## Table of Contents

1. [Switching Fundamentals](#1-switching-fundamentals)
2. [VLAN Segmentation & Inter-VLAN Routing](#2-vlan-segmentation--inter-vlan-routing)
3. [Dynamic Routing (Coming Soon)](#3-dynamic-routing-coming-soon)
4. [Access Control Lists (ACL) (Coming Soon)](#4-access-control-lists-acl-coming-soon)
5. [Firewall (Coming Soon)](#5-firewall-coming-soon)

---

## 1. Switching Fundamentals

*Source: Cisco - Basic Switching Concepts and Configuration*

### 1.1 Switch Boot & Management Prep

- **Boot Sequence**: POST → Boot loader → flash file system init → IOS image load → BOOT env variable check → NVRAM startup-config applied
- **System crash recovery**: Connect console cable → hold `Mode` button while reconnecting power → drops into `switch:` prompt
- **Minimum requirement for remote management**: assign IP/subnet mask to an SVI (Switch Virtual Interface); a default gateway is also needed if managing from a remote network
  → Management IP alone does **not** give Layer 3 routing capability (this ties directly into VLAN concepts, see [§2.1](#21-vlan-types))

### 1.2 Physical Layer Port Configuration

- **Duplex**: Full (simultaneous send/receive) vs Half (alternating)
- **Manual speed/duplex** vs **Auto-MDIX** (auto-detects cable type; when used, speed/ duplex must both be 'auto')
- Verify: 'show interfaces', `show controllers ethernet-controller ... | include Auto-MDIX`

### 1.3 Secure Remote Access - SSH

- Telnet (TCP 23, Plaintext)  → **SSH (TCP 22, encrypted)** is the recommended replacement
- Config order: 'ip domain-name' → 'crypto key generate rsa' → create local account → under 'line vty', set 'transport input ssh' + 'login local'
- Verify: 'show ip ssh', 'show ssh'
 → Naturally connects to restricting vty access to specific management IPs via ACL([§4 ACL, coming soon](#4-access-control-lists-acl-coming-soon))

### 1.4 Common LAN-Layer Attaks

| Attack | Mechanism | Defense |
|---|---|---|
| **MAC Address Flooding** | Flood bogus MACs → CAM table fills up → switch starts flooding all ports like a hub | **Port Security** |
| **DHCP Spoofing / Starvation** | Rogue DHCP server responds before the legitimate one → attacker impersonates default gateway | **DHCP Snooping** (trusted/untrusted port distinction) |

### 1.5 Switch Port Security

- Limits the number of allowed MAC addresses per port → on violation: **Violation Mode** = 'protect' / 'restrict' / 'shutdown' (default)
- Secure MAC registration methods: Static / Dynamic / **Sticky**
- On violation, port goes into **err-disabled** state → recover via 'shutdown' → ' no shutdown'
- Verify: 'show port-security interface', 'show port-security address'

- ### 1.6 Security Best Practices

- Written security policy, disable unused ports/services, strong passwords, physical access control, us HTTPS, regular backups, social engineering awareness training, encrypt sensitive data, implement firewalls, keep software updated
- → **DHCP Snooping** and **Port Security** are concrete implementations of these principles

- ---

## 2. VLAN Segmentation & Inter-VLAN Routing

*Source: Cisco - VLAN and Inter-VLAN Routing

### 2.1 VLAN Types

| Type | Role |
|---|---|
| Data VLAN | User-generated traffic |
| Default VLAN | All ports belong here initially (Cisco default = VLAN 1) |
| Native VLAN | Handles **untagged** frames on a trunk |
| Management VLAN | Used for switch management access (this is exactly what the SVI in [§1.1](#11-switch-boot--management-prep) gets its IP on) |

### 2.2 Purpose and Effect of VLANs
- Logical partition of a Layer 2 network  → **shrinks broadcast domains**
- Inter-VLAN communication is only possible **through a router**  → leads into [§2.4 Inter-VLAN Routing](#24-inter-vlan-routing)
- Benefits: security, cost reduction, better performance, management efficiency

### 2.3 VLAN Trunks & Tagging

- **Trunk link**: carries traffic VLANs over a single physical link(point-to-point)
- **802.1q Tagging**: inserts a header with Type (0x8100) + Priority + CFI + VID (12 bits)
- **Native VLAN rule**: untagged frames are treated as belonging to the native VLAN; everything else must be tagged.

### 2.4 Creating VLANs & Assigning Ports

- VLAN ranges: Normal (1-1005, stored in vlan.dat), EXtended (1006-4094,  stored in running-config)
- Verify: 'show vlan brief', 'show interfaces switchport'

### 2.5 Inter-VLAN Routing

Since VLANs are isolated from each other, a router is required for them to communicate - two approaches:

| Approach | Characteristics |
|---|---|
| **Legacy** | One **physical interface** per VLAN on the router → inefficient as VLAN count grows |
| **Router-on-a-Stick** | A single physical interface configured as an 802.1Q trunk → routing done via per-VLAN **subinterfaces** |

- Verify: 'show vlans", 'show ip route', 'ping'/'tracert'
→ The routing table built here later gets populated dynamically instead of statically in [§3 Dynamic Routing](#3-dynamic-routing-coming-soon)

---

## 3. Dynamic Routing

*Source: Cisco - Dynamic Routing Protocols

### 3.1 Dynamic Routing protocol Overview

- Routing protocols exchange routing information between routers to automate what static routes require manual configuration for discover remote networks, keep routing info current, pick the best path, and reconverge if the current path fails.
- Three core components: **data structures** (routing tables held in RAM), **routing protocol messages** (neighbor discovery, updates), **algorithm** (best-path calculation)
 
| | Interior Gateway (Distance Vector) | Interior Gateway (Link-State) | Exterior Gateway |
|---|---|---|---|
| IPv4 | RIPv2, EIGRP | OSPFv2, IS-IS | BGP-4 |
| IPv6 | RIPng, EIGRP for IPv6 | OSPFv3, IS-IS for IPv6 | BGP-MP |

### 3.2 Static vs Dynamic Routing

- Real networks mix both - static isn't obsolete, it's used for stub networks,  default routes, and small networks that won't grow
- Trade-off is explicit, not a matter of one being "better":

| | Static | Dynamic |
|---|---|---|
| Security | No advertisements sent — more secure by default | Advertisements sent — needs additional config to secure (ties to [§4 ACL](#4-access-control-lists-acl-coming-soon) restricting routing update sources) |
| Scaling | Config complexity grows fast with network size | Scales independently of network size |
| Resource use | No CPU/RAM overhead for algorithm/updates | Requires extra CPU, RAM, link bandwidth |
| Failure response | Manual re-route required | Reroutes automatically if a path fails |

→ This directly informs the trade-off discussion for the TMC network design write-up: justify *why* RIP was chosen over static routes for that topology, not just that it was configured.

### 3.3 Configuring RIPv2

**Enable RIP and advertise networks**


**Force version 2** (default sends v1, receives v1/v2 — must be set explicitly)
**Why RIPv2, not RIPv1 — VLSM/CIDR support**

RIPv1 is a **classful** protocol: it does not include the subnet mask in its routing updates. Every router assumes the mask based on the address class (A/B/C), so it cannot distinguish between different subnet sizes carved out of the same network — meaning it cannot support **VLSM** or **CIDR**.

- **VLSM (Variable-Length Subnet Masking)** — subnetting a network into subnets of *different* sizes (e.g. a /30 for a router-to-router link next to a /24 for a LAN), instead of one fixed mask across the whole network. Needed because a flat, equal-size subnet scheme wastes address space on point-to-point links.
- **CIDR (Classless Inter-Domain Routing)** — the general practice of using masks that ignore traditional class boundaries (A/B/C) and instead can be summarized or split at any bit boundary, letting routes be aggregated (e.g. advertising 192.168.0.0/22 instead of four separate /24s).

**RIPv2 is classless**: it carries the subnet mask alongside each route in its updates, so routers can correctly interpret VLSM'd subnets and CIDR-style summarized routes instead of guessing from address class.

→ This is exactly why [§3.3 `no auto-summary`](#33-configuring-ripv2) matters on top of just running v2: v2 supports carrying masks, but *auto-summary* still collapses routes back to their classful boundary at network edges unless you turn it off. If your TMC topology uses VLSM (which [§2.4](#24-creating-vlans--assigning-ports) subnetting almost certainly does), running v2 without disabling auto-summary will silently break reachability between non-contiguous subnets — that's a real failure mode worth screenshotting for the portfolio (misconfigured vs corrected `show ip route` output), not just a config step to tick off.

**Disable auto-summarization** (required for VLSM/discontiguous subnets - this is why it matters directly for [§2.4 VLAN subnetting](#24-creating-vlans--assigning-ports), where non-contiguous subnets per VLAN would otherwise get summarized incorrectly at classful boundaries) 

**Passive interface** - suppress RIP updates out of LAN-facing interfaces (the router still advertises the network, it just stops broadcasting update traffic onto that sement)

→ This is the RIP-specific instance of the [§1.6 Best Practice](#16-security-best-practices-10) "disable unused ports/services" principle: sending RIP updates onto an access-layer VLAN wastes bandwidth, wastes router/switch resources, and is a security exposure (any host on that segment can passively capture routing updates, or actively inject spoofed ones).

**Propagate a default route into RIP** (for a stub-to-internet edge router)

**Verification commands**

- `show ip protocols` confirms: version sent/received, auto-summary state, passive interfaces, routing sources with distance/last update
- `show ip route` confirms routes learned via RIP show as `R` with `[120/hop-count]` — administrative distance 120 is RIP's default, worth knowing cold if asked to compare AD across protocols in an interview

---
