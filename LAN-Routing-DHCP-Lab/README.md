# LAN Routing & Configuration Lab

A Cisco Packet Tracer lab demonstrating inter-VLAN/LAN routing, ICMP traversal across a router, and two different DHCP deployment methods - built to reinforce OSI Layer 2/3 concepts from the [Dissecting the OSI Layers](../dissecting_OSI_Layers)
series with a practical network build.

## Topology

Two separate LANs (192.168.10.0/24) and 192.168.11.0/24) connected via a single router (RT1), each with its own switch and end device.

*<img width="1019" height="726" alt="Screenshot 2026-08-12 135725" src="https://github.com/user-attachments/assets/546fa8a8-e7e5-44f0-92dd-96f6fe546c40" />*


| Device | Role | Network|
|---|---|---|
|RT1 (Cisco 1941) | Router - inter-LAN gateway | 10.1 / 11.1 |
| SW1 (2960-24TT) | Switch — LAN A | 192.168.10.0/24 |
| SW2 (2960-24TT) | Switch — LAN B | 192.168.11.0/24 |
| PC1 | End device — LAN A | 192.168.10.10 |
| PC2 | End device — LAN B | 192.168.11.10 |

## Activities

1. **[Static LAN Setup and Routing](./01-Static-LAN-Routing)** - manual IP configuration, ICMP ping/traceroute analysis across the router, Layer 2/3 PDU breakdown.
