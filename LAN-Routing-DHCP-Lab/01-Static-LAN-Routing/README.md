### Overview

This activity builds a two-LAN topology connected through a single router, with static IP addressing on both PCs. The goal is to observe how traffic crosses between two different subnets - specifically how Layer 2 (MAC) and Layer 3 (IP) addressing behave differently when the destination is outside the local subnet.

### Topology & Configuration

*<img width="1019" height="726" alt="Screenshot 2026-08-12 135725" src="https://github.com/user-attachments/assets/f835e85d-1323-4f1a-b579-5776366aadc1" />*

- **RT1** - GigabitEthernet0/0: '192.168.11.1/24', GigabitEthernet0/1: '192.168.10.1/24'
- **PC1** (LAN A): `192.168.10.10`, Gateway: `192.168.10.1` subnet mask: '255.255.255.0'
- **PC2** (LAN B): `192.168.11.10`, Gateway: `192.168.11.1` subnet mask: '255.255.255.0'
- 
<img width="1210" height="711" alt="Screenshot 2026-08-12 135959" src="https://github.com/user-attachments/assets/5f67f196-fc69-47e1-b616-671b6d267804" />
<img width="1232" height="712" alt="Screenshot 2026-08-12 135926" src="https://github.com/user-attachments/assets/f2abdd16-2e8d-48e5-8797-a56bf62e9aed" />

<img width="1029" height="718" alt="Screenshot 2026-08-12 140116" src="https://github.com/user-attachments/assets/c6a97797-d62d-49ca-b644-ec61d73317d6" />
<img width="1385" height="712" alt="Screenshot 2026-08-12 140556" src="https://github.com/user-attachments/assets/6384faa0-e1a7-4ff9-a3b3-1b75cb36dc91" />

---


### Lab / Practical

#### 1. ICMP Ping Across subnets

Pinged PC2 (`192.168.11.10`) from PC1 (`192.168.10.10`):

<img width="1537" height="926" alt="Screenshot 2026-08-12 141428" src="https://github.com/user-attachments/assets/01ee027d-f21c-4d91-9249-5ceb0f867948" />

The first request timing out is expected - it's consumed by the ARP resolution process (PC1 discovering the gateway's MAC's address) before the actual ICMP exchange can begin.


#### 2. Layer 2 Analysis - The Gateway's MAC, Not the Final Destination's

Using packet Tracer's Simulation Mode, inspected the Ethernet II header at each hop:


<img width="566" height="572" alt="Screenshot 2026-08-12 140906" src="https://github.com/user-attachments/assets/ee5633c7-16c6-41fd-b9a1-afcab3ec1856" />
<img width="1540" height="714" alt="Screenshot 2026-08-12 140956" src="https://github.com/user-attachments/assets/e0e264ff-15ab-4774-9bb8-6173bf40a883" />
<img width="1537" height="926" alt="Screenshot 2026-08-12 141428" src="https://github.com/user-attachments/assets/dd9e63d3-211e-464c-b162-dcca8cdd202d" />
<img width="1543" height="702" alt="Screenshot 2026-08-12 141533" src="https://github.com/user-attachments/assets/d417e587-41e1-4070-b83a-332676472b75" />
<img width="1543" height="701" alt="Screenshot 2026-08-12 141626" src="https://github.com/user-attachments/assets/92f9f2d5-b198-4ca1-9cb8-c7f1cc995366" />


| Hop | Destination MAC | Destination IP |
|---|---|---|
| PC1 → RT1 | `0007.ECEE.0201` (RT1's Gig0/1) | `192.168.11.10` (PC2, unchanged) |
| RT1 → PC2 | `000D.BDA1.2AD1` (PC2's actual MAC) | `192.168.11.10` (PC2, unchanged) |

This directly confirms the ARP behavior discussed in the Data Link Layer documentation: **the destination MAC address only ever points to the next hop(the gateway), never to the final destination**, while the destination IP address stays the same end-to-end.
The router rewrite the Layer 2 header at each hop while preserving the Layer 3 header - visible proof of the process described theoretically in the OSI series.

#### 3. Traceroute - Confirming the Path

<img width="1641" height="703" alt="Screenshot 2026-08-12 141808" src="https://github.com/user-attachments/assets/b53582e2-9ef1-44db-9a7f-b3ef6d6e7abe" />

The trace confirms traffic does **not** go directly from PC1 to PC2 - it must pass through the gateway('192.168.10.1', RT1's LAN a interface) first, since the two devices sit on different subnets.
This is the same TTL-based hop discovery mechanism documented in the Network Layer section, applied here to a small local topology instead of a public internet path.

---

### Key Takeaway

This lab makes visible, on a device-by-device basis, the concept previously explained in Layer 2/3 theory: crossing a subnet boundary always routes through a gateway, and each hop rewrites the Layer 2 header while the Layer 3 header(source/destination IP) remains unchanged for the entire journey.



