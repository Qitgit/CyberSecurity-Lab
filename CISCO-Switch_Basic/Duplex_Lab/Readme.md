#Overview

This lab demonstrates how hubs operate in a shared collision domain and how duplex settings affect communication reliability.
By sending ICMP echo requests between multiple PCs connected to a hub, we observe:

Broadcast-like behavior of hubs

Collisions when multiple devices transmit simultaneously

Differences between half duplex, full duplex, and auto duplex

How duplex mismatches impact network stability

##Topology##

<img width="1340" height="578" alt="Hub rep1" src="https://github.com/user-attachments/assets/c2b45fd0-5e39-4f47-8871-0613b9873fff" />

| Device | IP Address | Interface |
| --- | --- | --- |
| PC3 | 192.168.1.2 | Fa0 |
| PC4 | 192.168.1.3 | Fa0 |
| PC5 | 192.168.1.4 | Fa0 |
| Hub Hu1 | Shared Medium | Fa0/1/2 |
The hub operates in Half Duplex, meaning only one device can transmit at a time.


<img width="661" height="431" alt="Hub rep2" src="https://github.com/user-attachments/assets/53ae0f7e-8cee-44a4-a27d-a8f8293b2d66" />

<img width="642" height="427" alt="Hub rep3" src="https://github.com/user-attachments/assets/19dfb2fc-f3f3-48e9-9fa5-17e06cf4a761" />

###Ping from PC3 to PC4

What happens:

PC3 sends an ICMP Echo Request to PC4

Because the hub is a Layer 1 repeater, it copies the signal to all ports

So PC4 and PC5 both receive the ping frame

#Two PCs Sending Ping at the Same Time(collision)

<img width="1620" height="492" alt="Packet collsion1" src="https://github.com/user-attachments/assets/1ebee79f-751e-495c-927c-fa17d465fdcd" />
What happens:

PC3 sends ping to PC4

PC5 sends ping to PC3 at the same time

Both signals meet at the hub → collision occurs

<img width="654" height="428" alt="packet collsion2" src="https://github.com/user-attachments/assets/bd6cec74-2ca1-4e1c-bee3-a42fe45d1d55" />

Why this matters:

Half duplex = one device transmits at a time

When two devices transmit simultaneously → collision

Collisions cause retransmissions and packet loss

## Duplex Configuration on Switch 

<img width="1005" height="321" alt="3 dif duplex and speed mode" src="https://github.com/user-attachments/assets/2ccbbcfb-34c3-443d-bc7d-96067d54cc8d" />

#Configuration Steps#
Switch> enable
Switch# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)# interface FastEthernet0/21
Switch(config-if)# duplex full
Switch(config-if)# duplex half
Switch(config-if)# duplex auto
Switch(config-if)# speed ?
  10    Force 10 Mbps operation
  100   Force 100 Mbps operation
  auto  Enable AUTO speed configuration
Switch(config-if)# speed auto
*Key Insight:*  
Duplex and speed settings must match on both ends of a link.
A mismatch (e.g., one side full, the other half) can cause severe packet loss and unstable connections.

#Security Relevance###
Even though duplex configuration is a physical‑layer concept, it directly affects network reliability and security:

Availability (CIA Triad): Proper duplex settings ensure stable connectivity and prevent downtime.

Incident Response: Reliable links allow faster forensic data collection and monitoring.

Detection Accuracy: IDS/IPS sensors require full‑duplex links to capture all traffic without loss.

Operational Resilience: Auto‑negotiation prevents human error and maintains consistent performance.

##Result##

After configuration, both switches communicate efficiently with no collisions or retransmissions, confirming proper duplex negotiation and link stability.
