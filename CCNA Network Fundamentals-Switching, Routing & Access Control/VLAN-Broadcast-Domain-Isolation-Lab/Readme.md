# VLAN Broadcast Domain Isolation — Before/After Lab

Demonstrates that VLAN separation breaks connectivity **independent of IP subnetting** — two hosts on the same /24 subnet lose reachability once placed in different VLANs, with no router present to route between them.

Related concept notes: [Networking Concepts — §2 VLAN Segmentation & Inter-VLAN Routing](../CCNA%20Network%20Fundamentals-Switching%2C%20Routing%20%26%20Access%20Control/README.md#2-vlan-segmentation--inter-vlan-routing)

## Topology

- PC3 (192.168.1.11) — Fa0/3
- PC4 (192.168.1.12) — Fa0/4
- PC5 (192.168.1.23) — Fa0/5
- Switch2, single switch, no router in the topology
- All three hosts on **192.168.1.0/24** throughout — the subnet never changes

## Part 1 — Before VLAN Segmentation

All ports default to VLAN 1, so PC3 and PC5 are in the same broadcast domain.

![Initial VLAN brief](screenshots/01-initial-vlan-brief.png)
*All 24 ports on VLAN 1 (default). No VLANs configured yet.*

![Empty MAC table](screenshots/02-mac-table-empty.png)
*MAC address table empty before any traffic — switch hasn't learned any addresses yet.*

![Ping success before VLAN config](screenshots/03-ping-before-vlan-success.png)
*PC3 → PC5 ping succeeds: 4/4 received, 0% loss. Same VLAN, same subnet — no routing needed.*

![MAC table after ping](screenshots/04-mac-table-after-ping.png)
*Switch dynamically learned two entries — Fa0/3 (PC3) and Fa0/5 (PC5) — as a direct result of the ping traffic. Only these two ports appear because only these two hosts generated traffic; PC4 hasn't sent anything yet, so its MAC isn't in the table.*

## Part 2 — Creating and Assigning VLANs

```
Switch(config)# vlan 10
Switch(config-vlan)# name VLAN10
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name VLAN20
```

![VLANs created](screenshots/05-vlan10-20-created.png)
*VLAN 10 and VLAN 20 created and active, but no ports assigned yet — everything is still on VLAN 1 at this point. Note `vlan.dat` now appears in flash (676 bytes) — this is the VLAN database file referenced in the concept notes, created the moment a VLAN is defined.*

**Assigning a single port:**
```
Switch(config)# int f0/5
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
```
![Single port assignment](screenshots/06-port-assign-single.png)

**Assigning a port range:**
```
Switch(config)# int range f0/3 - 4
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
```
![Range port assignment](screenshots/07-port-assign-range.png)

![VLAN brief after assignment](screenshots/08-vlan-brief-after-assignment.png)
*Confirms the split: VLAN 10 → Fa0/3, Fa0/4 (PC3, PC4). VLAN 20 → Fa0/5 (PC5). Remaining ports stay on default VLAN 1.*

## Part 3 — After VLAN Segmentation

![Ping fails after VLAN config](screenshots/09-ping-after-vlan-fail.png)
*Same command, same source, same destination, same subnet (192.168.1.11 → 192.168.1.23) — now 4/4 lost, 100% loss.*

## Why This Happens

PC3 and PC5's IP addresses didn't change. Their subnet mask didn't change. The only thing that changed is which VLAN their switch port belongs to. VLANs create **separate broadcast domains at Layer 2**, and a switch will not forward frames between VLANs regardless of what IP addressing looks like on top — that's a Layer 3 (routing) job. Since this topology has no router, no SVI, and no Layer 3 device of any kind, there is nothing to route between VLAN 10 and VLAN 20, so the ping has no path to complete.

Fixing this requires either:
- A router doing **Inter-VLAN Routing** (Legacy or Router-on-a-Stick) — see [main concept notes §2.5](../CCNA%20Network%20Fundamentals-Switching%2C%20Routing%20%26%20Access%20Control/README.md#25-inter-vlan-routing)
- Or putting both ports back in the same VLAN, if isolation was never the intent

## Commands Used

```
show vlan brief
show mac address-table
ping <ip>
vlan <id>
name <vlan-name>
interface <if>
switchport mode access
switchport access vlan <id>
interface range <if-range>
```
