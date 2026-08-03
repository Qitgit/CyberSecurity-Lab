### Overview

The Transport Layer is Layer 4 of the OSI model. While Layer 3 (Network) gets packets from one device to another across networks using IP addresses,Layer 4 is reponsible for getting data to the **correct application** on that device, and (depending on the protocol) ensuring it arrives reliably and in order.

Key responsibilities of this layer:

- **Port addressing** : identifying which application/service on a device should receive the data(e.g., port 443 for HTTPS, port 53 for DNS)
- **Segmentatoin**: breaking data from the application layer into smaller units(segments for TCP, datagrams for UDP) for trasmission
- **reliability (TCP only)**: ensuring data arrives completely, in order, and without correption - via acknowledgments, retransmission, and sequencing
- **Flow Control**: preventing a fast sender from overwhelming a slow receiver
- ** Connection Management (TCP only)**: establish and tearing down connections via the 3-way handshake / 4-way terminaiton


### TCP vs UDP
|---| TCP | UDP|

| Connection | Connetion-oriented (handshake required) | Connetionless (no handshake)
| Reliability | Guaranteed delivery, retransmits lost data | No guarantee, "fire and forget" |
| Ordering | Data arrives in order | No ordering guarantee |
| Speed | Slower (overhead from reliabilty checks) | Faster (minimal overhead) |
| Use cases | Web browsing (HTTPS), email, file transfer | DNS, Video streaming, VoIP, gaming | 

The choice between TCP and UDP is a trade-off: **TCP prioritizes reliability, UDP prioritizes speed.**

Applications where losing a packet is worse than delay (e.g., loading a webpage) use TCP; application where delay is worse than losing a packet (e.g., live video call) use UDP.

### TCP 3-Way Handshake Capture

Captured a live TCP connection establishment using Wireshark:

