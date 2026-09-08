# Cisco Networking Concepts — Switching, Routing & Access Control

> Concept notes based on the CCNA (Routing and Switching Essentials) curriculum.
> Order: Switching Fundamentals → VLAN → Dynamic Routing → ACL → Firewall.
> Each concept is cross-linked so it maps 1:1 to a Packet Tracer lab.

---

## Table of Contents

1. [Switching Fundamentals](#1-switching-fundamentals)
2. [VLAN Segmentation & Inter-VLAN Routing](#2-vlan-segmentation--inter-vlan-routing)
3. [Dynamic Routing](#3-dynamic-routing)
4. [Access Control Lists (ACL)](#4-access-control-lists-acl)
5. [Firewall](#5-firewall)
6. [Lab Checklist](#6-lab-checklist)

---

## 1. Switching Fundamentals

*Source: Cisco – Basic Switching Concepts and Configuration*

### 1.1 Switch Boot & Management Prep

- **Boot Sequence**: POST → Boot loader → flash file system init → IOS image load → BOOT env variable check → NVRAM startup-config applied
- **System crash recovery**: Connect console cable → hold `Mode` button while reconnecting power → drops into `switch:` prompt
- **Minimum requirement for remote management**: assign IP/subnet mask to an SVI (Switch Virtual Interface); a default gateway is also needed if managing from a remote network
  → Management IP alone does **not** give Layer 3 routing capability (this ties directly into VLAN concepts, see [§2.1](#21-vlan-types))

```
S1(config)# interface vlan99
S1(config-if)# ip address 172.17.99.11 255.255.255.0
S1(config-if)# no shutdown
S1(config)# ip default-gateway 172.17.99.1
```

### 1.2 Physical Layer Port Configuration

- **Duplex**: Full (simultaneous send/receive) vs Half (alternating)
- **Manual speed/duplex** vs **Auto-MDIX** (auto-detects cable type; when used, speed/duplex must both be `auto`)
- Verify: `show interfaces`, `show controllers ethernet-controller ... | include Auto-MDIX`

### 1.3 Secure Remote Access — SSH

- Telnet (TCP 23, plaintext) → **SSH (TCP 22, encrypted)** is the recommended replacement
- Config order: `ip domain-name` → `crypto key generate rsa` → create local account → under `line vty`, set `transport input ssh` + `login local`
- Verify: `show ip ssh`, `show ssh`
  → Restricting vty access to specific management IPs is a **standard ACL** job — see [§4.4](#44-standard-vs-extended-acls)

### 1.4 Common LAN-Layer Attacks

| Attack | Mechanism | Defense |
|---|---|---|
| **MAC Address Flooding** | Flood bogus MACs → CAM table fills up → switch starts flooding all ports like a hub | **Port Security** |
| **DHCP Spoofing / Starvation** | Rogue DHCP server responds before the legitimate one → attacker impersonates default gateway | **DHCP Snooping** (trusted/untrusted port distinction) |

### 1.5 Switch Port Security

- Limits the number of allowed MAC addresses per port → on violation: **Violation Mode** = `protect` / `restrict` / `shutdown` (default)
- Secure MAC registration methods: Static / Dynamic / **Sticky**
- On violation, port goes into **err-disabled** state → recover via `shutdown` → `no shutdown`
- Verify: `show port-security interface`, `show port-security address`

### 1.6 Security Best Practices (10)

Written security policy, disable unused ports/services, strong passwords, physical access control, use HTTPS, regular backups, social engineering awareness training, encrypt sensitive data, implement firewalls, keep software updated
→ **DHCP Snooping** and **Port Security** are concrete implementations of these principles

---

## 2. VLAN Segmentation & Inter-VLAN Routing

*Source: Cisco – VLAN and Inter-VLAN Routing (Chapter 6)*

### 2.1 VLAN Types

| Type | Role |
|---|---|
| Data VLAN | User-generated traffic |
| Default VLAN | All ports belong here initially (Cisco default = VLAN 1) |
| Native VLAN | Handles **untagged** frames on a trunk |
| Management VLAN | Used for switch management access (this is exactly what the SVI in [§1.1](#11-switch-boot--management-prep) gets its IP on) |

### 2.2 Purpose and Effect of VLANs

- Logical partition of a Layer 2 network → **shrinks broadcast domains**
- Inter-VLAN communication is only possible **through a router** → leads into [§2.5 Inter-VLAN Routing](#25-inter-vlan-routing)
- Benefits: security, cost reduction, better performance, management efficiency

### 2.3 VLAN Trunks & Tagging

- **Trunk link**: carries traffic for multiple VLANs over a single physical link (point-to-point)
- **802.1Q Tagging**: inserts a header with Type (0x8100) + Priority + CFI + VID (12 bits)
- **Native VLAN rule**: untagged frames are treated as belonging to the native VLAN; everything else must be tagged

```
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 99
S1(config-if)# switchport trunk allowed vlan 10,20,30,99
```

### 2.4 Creating VLANs & Assigning Ports

```
S1(config)# vlan 20
S1(config-vlan)# name student
S1(config)# interface f0/18
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 20
```

- VLAN ranges: Normal (1–1005, stored in vlan.dat), Extended (1006–4094, stored in running-config)
- Verify: `show vlan brief`, `show interfaces switchport`

### 2.5 Inter-VLAN Routing

Since VLANs are isolated from each other, a router is required for them to communicate — two approaches:

| Approach | Characteristics |
|---|---|
| **Legacy** | One **physical interface** per VLAN on the router → inefficient as VLAN count grows |
| **Router-on-a-Stick** | A single physical interface configured as an 802.1Q trunk → routing done via per-VLAN **subinterfaces** |

```
R1(config)# interface g0/0.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 172.17.10.1 255.255.255.0
```

- Verify: `show vlans`, `show ip route`, `ping`/`tracert`
→ The routing table built here later gets populated dynamically instead of statically in [§3 Dynamic Routing](#3-dynamic-routing). VLAN-to-VLAN traffic filtering (e.g. blocking one department's LAN from reaching another) is an **extended ACL** job — see [§4.6](#46-configuring-an-extended-acl).

---

## 3. Dynamic Routing

*Source: Cisco – Dynamic Routing Protocols (Ch. 3)*

### 3.1 Dynamic Routing Protocol Overview

- Routing protocols exchange routing information between routers to automate what static routes require manual configuration for: discover remote networks, keep routing info current, pick the best path, and reconverge if the current path fails
- Three core components: **data structures** (routing tables held in RAM), **routing protocol messages** (neighbor discovery, updates), **algorithm** (best-path calculation)

| | Interior Gateway (Distance Vector) | Interior Gateway (Link-State) | Exterior Gateway |
|---|---|---|---|
| IPv4 | RIPv2, EIGRP | OSPFv2, IS-IS | BGP-4 |
| IPv6 | RIPng, EIGRP for IPv6 | OSPFv3, IS-IS for IPv6 | BGP-MP |

### 3.2 Static vs Dynamic Routing

- Real networks mix both — static isn't obsolete, it's used for stub networks, default routes, and small networks that won't grow
- Trade-off is explicit, not a matter of one being "better":

| | Static | Dynamic |
|---|---|---|
| Security | No advertisements sent — more secure by default | Advertisements sent — needs additional config to secure |
| Scaling | Config complexity grows fast with network size | Scales independently of network size |
| Resource use | No CPU/RAM overhead for algorithm/updates | Requires extra CPU, RAM, link bandwidth |
| Failure response | Manual re-route required | Reroutes automatically if a path fails |

→ This directly informs the trade-off discussion for the TMC network design write-up: justify *why* RIP was chosen over static routes for that topology, not just that it was configured.

### 3.3 Configuring RIPv2

**Enable RIP and advertise networks**

```
R1(config)# router rip
R1(config-router)# network 192.168.1.0
R1(config-router)# network 192.168.2.0
```

**Why RIPv2, not RIPv1 — VLSM/CIDR support**

RIPv1 is a **classful** protocol: it does not include the subnet mask in its routing updates. Every router assumes the mask based on the address class (A/B/C), so it cannot distinguish between different subnet sizes carved out of the same network — meaning it cannot support **VLSM** or **CIDR**.

- **VLSM (Variable-Length Subnet Masking)** — subnetting a network into subnets of *different* sizes (e.g. a /30 for a router-to-router link next to a /24 for a LAN), instead of one fixed mask across the whole network. Needed because a flat, equal-size subnet scheme wastes address space on point-to-point links.
- **CIDR (Classless Inter-Domain Routing)** — the general practice of using masks that ignore traditional class boundaries (A/B/C) and instead can be summarized or split at any bit boundary, letting routes be aggregated (e.g. advertising 192.168.0.0/22 instead of four separate /24s).

RIPv2 is **classless**: it carries the subnet mask alongside each route in its updates, so routers can correctly interpret VLSM'd subnets and CIDR-style summarized routes instead of guessing from address class.

**Force version 2** (default sends v1, receives v1/v2 — must be set explicitly)

```
R1(config)# router rip
R1(config-router)# version 2
```

**Disable auto-summarization** (required for VLSM/discontiguous subnets — this is why it matters directly for [§2.4 VLAN subnetting](#24-creating-vlans--assigning-ports), where non-contiguous subnets per VLAN would otherwise get summarized incorrectly at classful boundaries)

```
R1(config-router)# no auto-summary
```

→ v2 supports carrying masks, but *auto-summary* still collapses routes back to their classful boundary at network edges unless you turn it off. If the TMC topology uses VLSM (which [§2.4](#24-creating-vlans--assigning-ports) subnetting almost certainly does), running v2 without disabling auto-summary will silently break reachability between non-contiguous subnets — that's a real failure mode worth screenshotting for the portfolio (misconfigured vs corrected `show ip route` output), not just a config step to tick off.

**Passive interfaces** — suppress RIP updates out of LAN-facing interfaces (the router still advertises the network, it just stops broadcasting update traffic onto that segment)

```
R1(config-router)# passive-interface g0/0
```

→ This is the RIP-specific instance of the [§1.6 Best Practice](#16-security-best-practices-10) "disable unused ports/services" principle: sending RIP updates onto an access-layer VLAN wastes bandwidth, wastes router/switch resources, and is a security exposure (any host on that segment can passively capture routing updates, or actively inject spoofed ones).

**Propagate a default route into RIP** (for a stub-to-internet edge router)

```
R1(config)# ip route 0.0.0.0 0.0.0.0 S0/0/1 209.165.200.226
R1(config)# router rip
R1(config-router)# default-information originate
```

**Verification commands**

```
show ip protocols
show ip route | begin Gateway
```

- `show ip protocols` confirms: version sent/received, auto-summary state, passive interfaces, routing sources with distance/last update
- `show ip route` confirms routes learned via RIP show as `R` with `[120/hop-count]` — administrative distance 120 is RIP's default, worth knowing cold if asked to compare AD across protocols in an interview

---

## 4. Access Control Lists (ACL)

*Source: Cisco – Access Control Lists (Ch. 4)*

### 4.1 What an ACL Does

- An ACL is a sequential list of permit/deny statements called **ACEs (Access Control Entries)**
- As traffic hits an interface with an ACL applied, the router checks it against each ACE **top to bottom** and stops at the first match — order matters, this is not evaluated as a whole set
- **Every ACL has an implicit `deny any` at the end.** If nothing matches, the packet is dropped — this is why [§4.5](#45-configuring-a-standard-acl) explicitly adds `permit any` after a `deny host`: without it, that ACL would silently block everything, not just the one host.

### 4.2 Wildcard Masks

- A wildcard mask is the **inverse logic** of a subnet mask: `0` = must match this bit, `1` = ignore this bit
- This is the same VLSM math from [§3.3](#33-configuring-ripv2) run backwards — instead of "which bits identify the subnet," it's "which bits do I care about matching"

| IP Address | Wildcard | Result | Meaning |
|---|---|---|---|
| 192.168.1.1 | 0.0.0.0 | 192.168.1.1 | Match this exact host |
| 192.168.1.1 | 255.255.255.255 | 0.0.0.0 | Match **any** host (equivalent to keyword `any`) |
| 192.168.1.1 | 0.0.0.255 | 192.168.1.0 | Match the whole /24 subnet |

- Shortcut keywords: `host 192.168.1.1` = `192.168.1.1 0.0.0.0`; `any` = `0.0.0.0 255.255.255.255`

### 4.3 Applying ACLs to an Interface

- **Inbound**: filters packets before they're routed. **Outbound**: filters after routing, regardless of which interface they came in on.
- Hard rule: **one ACL per protocol, per direction, per interface.** Two interfaces × two protocols (IPv4/IPv6) = up to 8 separate ACLs on one router.

### 4.4 Standard vs Extended ACLs

| | Standard | Extended |
|---|---|---|
| Filters on | Source IP address only | Source IP, destination IP, protocol, source/destination port |
| Number range | 1–99, 1300–1999 | 100–199, 2000–2699 |
| Placement | As close to the **destination** as possible (it can't distinguish traffic by where it's going, so filtering early would block more than intended) | As close to the **source** as possible (specific enough to filter precisely right where the traffic originates, before it consumes bandwidth elsewhere) |

→ This placement rule is the direct answer to the open question sitting in [§1.3](#13-secure-remote-access--ssh): restricting SSH/vty access to specific management IPs is a **standard** ACL job (matches on source only) — apply it close to the vty lines, not out on the edge.

**Named ACLs** — alternative to numbers: alphanumeric name, must be unique, cannot start with a number, cannot contain spaces/punctuation, and unlike numbered ACLs you can insert/delete individual entries by sequence number without rebuilding the whole list (see [§4.7](#47-editing-acls)).

### 4.5 Configuring a Standard ACL

**Numbered:**
```
R1(config)# access-list 10 remark Permit hosts from the 192.168.10.0 LAN
R1(config)# access-list 10 permit 192.168.10.0 0.0.0.255
R1(config)# interface s0/0/0
R1(config-if)# ip access-group 10 out
```

**Named:**
```
R1(config)# ip access-list standard NO_ACCESS
R1(config-std-nacl)# deny host 192.168.11.10
R1(config-std-nacl)# permit any
R1(config-std-nacl)# exit
R1(config)# interface g0/0
R1(config-if)# ip access-group NO_ACCESS out
```

**Verify:**
```
show access-lists
show ip interface g0/0
```
`show ip interface` confirms which ACL is active per direction (`Outgoing access list is NO_ACCESS`) — this is the fast way to check if an ACL is actually applied, not just configured. Configuring an ACL and forgetting `ip access-group` is a common gap — the list exists but does nothing.

### 4.6 Configuring an Extended ACL

```
R1(config)# access-list 103 permit tcp 192.168.10.0 0.0.0.255 any eq 80
R1(config)# access-list 103 permit tcp 192.168.10.0 0.0.0.255 any eq 443
R1(config)# access-list 104 permit tcp any 192.168.10.0 0.0.0.255 established
R1(config)# interface g0/0
R1(config-if)# ip access-group 103 in
R1(config-if)# ip access-group 104 out
```
- `established` matches only TCP replies (ACK/RST flag set) — used to permit return traffic for connections initiated from inside, without opening the interface to unsolicited inbound connections
- Named extended ACLs work the same way, just under `ip access-list extended NAME`, and matter here because they're self-documenting — `SURFING` / `BROWSING` reads better in a `show access-lists` output during an interview than `access-list 103` does

**Applying an extended ACL to restrict traffic between departments** (the [§2.5](#25-inter-vlan-routing) VLAN-to-VLAN traffic filtering case):
```
R1(config)# access-list 101 deny tcp 192.168.11.0 0.0.0.255 192.168.10.0 0.0.0.255 eq ftp
R1(config)# access-list 101 deny tcp 192.168.11.0 0.0.0.255 192.168.10.0 0.0.0.255 eq ftp-data
R1(config)# access-list 101 permit ip any any
R1(config)# interface g0/1
R1(config-if)# ip access-group 101 in
```
Note the explicit `permit ip any any` at the end — without it, the implicit deny from [§4.1](#41-what-an-acl-does) would block **all** traffic from that VLAN, not just FTP.

### 4.7 Editing ACLs

Two ways, don't mix them up:
- **Text editor method**: `no access-list <number>` removes the whole list, then paste the corrected version back in — destructive, all-or-nothing
- **Sequence number method** (named ACLs only): `no <seq#>` removes just that one line, then re-add with the same sequence number to keep it in position
```
R1(config)# ip access-list extended SURFING
R1(config-ext-nacl)# no 10
R1(config-ext-nacl)# 10 permit tcp 192.168.10.0 0.0.0.255 any eq www
```
This matters operationally: on a live ACL controlling production traffic, `no access-list` drops the whole filter for the seconds between removal and re-entry. Sequence numbers avoid that gap.

---

## 5. Firewall

*Source: Cisco – Implementing Firewall Technologies (Ch. 4)*

### 5.1 What a Firewall Is

All firewalls share three properties: resistant to attack, the **only transit point** between networks (all traffic must pass through it), and enforce an access control policy.

### 5.2 Antispoofing with ACLs

An external-facing interface should never accept inbound traffic claiming to come from private, loopback, multicast, or broadcast address space — no legitimate external host uses those as a source address, so seeing one means the source is spoofed.

```
R1(config)# access-list 150 deny ip 0.0.0.0 255.255.255.255 any
R1(config)# access-list 150 deny ip 10.0.0.0 0.255.255.255 any
R1(config)# access-list 150 deny ip 127.0.0.0 0.255.255.255 any
R1(config)# access-list 150 deny ip 172.16.0.0 0.15.255.255 any
R1(config)# access-list 150 deny ip 192.168.0.0 0.0.255.255 any
R1(config)# access-list 150 deny ip 224.0.0.0 15.255.255.255 any
R1(config)# access-list 150 deny ip host 255.255.255.255 any
```
Applied inbound on the internet-facing interface. Mirror rule on the internal interface only permits traffic actually sourced from the internal subnet:
```
R1(config)# access-list 105 permit ip 192.168.1.0 0.0.0.255 any
```
→ This is a direct application of [§4.6 extended ACL syntax](#46-configuring-an-extended-acl) — nothing new here syntactically, just a specific, well-known rule set. Worth knowing this exists as a named pattern ("antispoofing" / "bogon filtering") for an interview, not just as ACL lines.

### 5.3 Permitting Necessary Traffic Through a Firewall

Default-deny, then explicitly permit only what's needed — inbound only to the specific server and port that needs it, and admin protocols (SSH, syslog, SNMP trap) only from the specific admin host, not from `any`:
```
R1(config)# access-list 180 permit udp any host 192.168.20.2 eq domain
R1(config)# access-list 180 permit tcp any host 192.168.20.2 eq smtp
R1(config)# access-list 180 permit tcp any host 192.168.20.2 eq ftp
R1(config)# access-list 180 permit tcp host 200.5.5.5 host 10.0.1.1 eq 22
R1(config)# access-list 180 permit udp host 200.5.5.5 host 10.0.1.1 eq syslog
R1(config)# access-list 180 permit udp host 200.5.5.5 host 10.0.1.1 eq snmptrap
```
→ This is the practical form of the [§4.1](#41-what-an-acl-does) implicit deny principle: don't write a deny-everything-except rule, write only the permits you need and let the implicit deny do the rest.

### 5.4 Mitigating ICMP Abuse

ICMP is useful for diagnostics but abusable for reconnaissance (ping sweeps) and DoS. Don't block it wholesale — permit only the ICMP types that are actually needed in each direction:
```
! Inbound from Internet — only allow replies to pings WE sent out
R1(config)# access-list 112 permit icmp any any echo-reply
R1(config)# access-list 112 permit icmp any any source-quench
R1(config)# access-list 112 permit icmp any any unreachable
R1(config)# access-list 112 deny icmp any any
R1(config)# access-list 112 permit ip any any

! Outbound from internal LAN — allow us to ping out, but not much else
R1(config)# access-list 114 permit icmp 192.168.1.0 0.0.0.255 any echo
R1(config)# access-list 114 permit icmp 192.168.1.0 0.0.0.255 any parameter-problem
R1(config)# access-list 114 permit icmp 192.168.1.0 0.0.0.255 any packet-too-big
R1(config)# access-list 114 permit icmp 192.168.1.0 0.0.0.255 any source-quench
R1(config)# access-list 114 deny icmp any any
R1(config)# access-list 114 permit ip any any
```
Note `deny icmp any any` is immediately followed by `permit ip any any` in both — without that last line, the [§4.1](#41-what-an-acl-does) implicit deny would silently block **all** IP traffic, not just the unwanted ICMP types.

### 5.5 Types of Firewalls (by OSI Layer Coverage)

| Type | Layers | What it does |
|---|---|---|
| **Packet Filtering** | L3–L4 | Matches source/dest IP, protocol, source/dest port, SYN flag — this is exactly what [§4 standard/extended ACLs](#4-access-control-lists-acl) already do |
| **Stateful** | L3–L5 | Adds a **state table** tracking established sessions — only permits return traffic that matches a session it actually saw initiated |
| **Application Gateway (Proxy)** | L3–L7 | Terminates and re-originates the connection itself, inspecting all the way up to the application layer |
| **NAT** | L3–L4 | Hides internal addressing as a side effect of translation |

### 5.6 Stateful Firewalls vs Plain ACLs

This is the answer to a gap [§4.6](#46-configuring-an-extended-acl) glossed over: the `established` keyword on an extended ACL is **not real state tracking** — it just checks whether the ACK or RST flag is set on an inbound TCP segment. It's a stateless approximation: an attacker can simply set the ACK flag on a crafted packet and get past it. A real stateful firewall maintains a **state table** built from watching the actual TCP handshake (or UDP/ICMP session data) and only lets return traffic through that matches a session it saw begin.

| Benefits | Limitations |
|---|---|
| Primary means of defense | No application-layer inspection |
| Strong packet filtering | Cannot filter stateless protocols |
| Better performance than static packet filters | Struggles with dynamic port negotiation (e.g. FTP active mode, VoIP) |
| Defends against spoofing and DoS | No user authentication support |
| Richer logging | |

**Next-Generation Firewalls (NGFW)** go further: app-level visibility/control, reputation-based web filtering, policy enforcement by user/device/role/app/threat profile, plus NAT, VPN, and IPS in one device.

### 5.7 Network Zones

- **Inside (trusted/private)** vs **Outside (untrusted/public)**: inside can initiate out (HTTP/SMTP/DNS), outside gets no unsolicited access in — this is the plain-language version of the [§5.3](#53-permitting-necessary-traffic-through-a-firewall) permit rules above
- **DMZ**: a middle zone for the servers that legitimately need to be reached from the internet (web/mail/DNS), so the private network never has to be directly exposed. Traffic pattern: DMZ ↔ Internet is selectively permitted, Internet → Private is blocked outright, Private → Internet/DMZ is inspected and mostly permitted.
- **Zone-Based Policy Firewall (ZPF)**: interfaces are grouped into named zones; traffic between two interfaces in the **same** zone flows freely with no policy applied by default — policy only gets enforced on traffic crossing **between** zones. This is a materially different model from interface-based ACLs (which apply per-interface, not per-zone-pair) and is what modern Cisco firewall configs (ASA, IOS ZFW) actually use instead of the flat `ip access-group` model in [§4](#4-access-control-lists-acl).

### 5.8 Layered Defense & Best Practices

Firewalls are one layer, not the whole defense: network core security, perimeter security, endpoint security, and communications security all need to exist alongside it.

Firewall-specific best practices:
- Position firewalls at actual security boundaries (not arbitrarily)
- Don't rely on the firewall exclusively — defense in depth
- **Deny all by default, permit only what's needed** — the underlying principle behind every ACL in [§4](#4-access-control-lists-acl) and every rule in this section
- Control physical access to the firewall itself
- Monitor firewall logs, not just configure and forget
- Use change management for firewall config changes — an unreviewed rule change is how misconfigurations ship to production
- Remember firewalls primarily stop **technical attacks from outside** — they don't stop insider threats, social engineering, or an already-compromised internal host

---

## 6. Lab Checklist

- [x] Switch boot/management IP setup (SVI)
- [x] SSH remote access configuration
- [x] Port Security (Sticky) configuration + violation test
- [x] DHCP Snooping trusted/untrusted port setup
- [x] VLAN creation and port assignment
- [x] 802.1Q trunk configuration (with native VLAN specified)
- [x] Legacy Inter-VLAN Routing
- [x] Router-on-a-Stick Inter-VLAN Routing
- [ ] RIPv2 configuration verified with `show ip route` / `show ip protocols` screenshots
- [ ] RIPv1 vs RIPv2 + `no auto-summary` before/after comparison (VLSM failure demo)
- [ ] OSPF configuration (coming soon)
- [ ] Standard ACL restricting vty (SSH) access to a management subnet
- [ ] Extended ACL filtering VLAN-to-VLAN traffic (e.g. block FTP between departments)
- [ ] `show ip interface` verification screenshot proving the ACL is actually applied, not just configured
- [ ] Antispoofing ACL on the WAN-facing interface (bogon/private-range filtering)
- [ ] Explicit permit rules for necessary services only, default-deny confirmed via `show access-lists`
- [ ] ICMP filtering — permit only required types, explicit `permit ip any any` after the ICMP deny
- [ ] Firewall / zone-based policy — implementation still pending (Cisco IOS ZFW or ASA config, if TMC scope covers it)

---

*Note: Source material based on Cisco Networking Academy content. This document is a study summary / portfolio artifact; lab screenshots and .pkt files will be linked separately.*
