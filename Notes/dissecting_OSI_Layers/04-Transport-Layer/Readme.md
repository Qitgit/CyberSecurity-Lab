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

##Lab / Practical

### TCP 3-Way Handshake Capture

Captured a live TCP connection establishment using Wireshark:

<img width="1119" height="282" alt="TCP 3 way handshake" src="https://github.com/user-attachments/assets/2778e3fb-9c08-423b-aa15-a86adb0400dd" />

1. **Packet 300(SYN)**: client initiates connection, proposing initial sequence number 'Seq=0'
2. **Packet 308 (SYN, ACK)**: server acknowledges the client's SYN ('Ack=1') and sends its own ('Seq=0') - this is called the the "SYN-ACK"
3. **Packet 309 (ACK)**: client acknowledges the server's SYN ('Ack=1') - handshake complete, connection established

After the handshake, the capture shows the TLS negotiation beginning immediately (Client Hello > Server Hello > Encrypted Handshake) - this is the application layer establishing an encrypted session on top of the now-reliable TCP connection.

### UDP / DNS Query Capture

Filtered on 'udp.port==53' while resolving a domain name:

<img width="969" height="80" alt="UDP,DNS capture" src="https://github.com/user-attachments/assets/62091a72-e47c-47aa-8633-f22e9f5198d5" />

Unlike TCP's 3-way handshake, DNS over UDP requires no connection setup - the client sends a single query, and the server replies with a single response. This reflects UDP's connectionless, "fire and forget" nature: there's no handshake, no exchange, which is ideal for a *quick* lookup like this.


**Note**: both the query (785) and response (786) share same transaction ID - this is how the client matches the response back to its original  request, since UDP has no concept of sequence numbers to track message order.


### Checking Active Connections with `netstat -ano`

<img width="677" height="737" alt="netstat " src="https://github.com/user-attachments/assets/dbdd67cc-2ae8-4d87-9380-6cd94767e226" />

- **Local Address**: always shows this machine's own IP (`192.168.1.199`) — 
  netstat only reports connections belonging to processes running on this host
- **Foreign Address**: the remote server's IP:port for each connection — mostly 
  port `443` (HTTPS), confirming most traffic is encrypted web traffic
- **State**: `ESTABLISHED` (active connection), `TIME_WAIT` (recently closed, 
  waiting to fully release), `LISTENING` (a local service waiting for incoming 
  connections)
- **Well-known ports observed**: `445` (SMB), `135` (RPC) — both listening 
  locally, standard Windows services
- **Ephemeral ports observed**: `49664`–`49669` — dynamically assigned by the OS 
  for outbound client connections, consistent with the ephemeral port range 
  (typically 49152–65535)

This shows the practical difference between **well-known ports** (fixed, 
service-specific, e.g., 443 for HTTPS) and **ephemeral ports** (temporary, 
randomly assigned per connection).

### Protocol Stack in a Single Packet

Expanding packet 785 reveals every OSI layer stacked within a single frame:

<img width="889" height="126" alt="UDP Capture" src="https://github.com/user-attachments/assets/fa0bd479-7183-4721-8a44-61d22cffefe4" />

- **Ethernet II** (Layer 2): frame header with source/destination MAC
- **Internet Protocol Version 4** (Layer 3): IP header with source/destination IP
- **User Datagram Protocol** (Layer 4): `Src Port: 53233` (ephemeral port chosen 
  by the client), `Dst Port: 53` (well-known port for DNS)
- **Domain Name System** (Layer 7): the actual DNS query data

This is a practical example of **encapsulation** — each layer wraps the layer 
above it with its own header before transmission.

### Checking Active Connections with `netstat -ano`

*(insert screenshot)*

- **Local Address**: always this machine's own IP (`192.168.1.199`) — netstat 
  only reports connections belonging to processes running on this host
- **Foreign Address**: the remote server's IP:port — mostly port `443` (HTTPS)
- **State**: `ESTABLISHED` (active), `TIME_WAIT` (recently closed), `LISTENING` 
  (a local service waiting for incoming connections)
- **Well-known ports observed**: `445` (SMB), `135` (RPC) — standard Windows 
  services listening locally
- **Ephemeral ports observed**: `49664`–`49669` — dynamically assigned by the OS 
  for outbound connections (typical range: 49152–65535)

This demonstrates the practical difference between **well-known ports** (fixed, 
service-specific) and **ephemeral ports** (temporary, randomly assigned per 
connection).


## Security Perspective

### Port Scanning

Attackers probe a target to discover which ports are open before planning further attacks. Several techniques exploit how TCO flags behave :

_ **TCP SYN Scan ("Half-open scan")**: sends a SYN packet like a normal handshake attempt, but never completes it
- Port **open**: target responds with SYN/ACK (same as packet 308 in our capture)
  - Port **closed**: target responds with RST/ACK instead
  - This is "stealthy" because a full connection is never established — it may not 
    get logged by the application, only at the network/OS level

    
- **TCP Connect Scan**: completes the full 3-way handshake — slower and easier to 
  detect (application-level logs will show the connection), but doesn't require 
  raw socket privileges

  
- **UDP Scan**: since UDP has no handshake, scanners send a UDP packet and wait — 
  no response often means the port *might* be open (or just silently dropped), 
  while an ICMP "Port Unreachable" response confirms the port is closed. This makes 
  UDP scanning inherently slower and less reliable than TCP scanning.

**Detection**: a burst of SYN packets across many different destination ports 
from the same source, especially without completing handshakes, is a strong 
indicator of port scanning. This is exactly the kind of pattern the 
`tcp.flags.syn==1` filter used earlier in this lab would help an analyst isolate 
during an investigation.

---

### 2. SYN Flood (DoS Attack)

Exploits the fact that a server must allocate resources (a half-open connection 
entry) as soon as it receives a SYN — before the handshake even completes.

**Attack flow:**
1. Attacker sends a massive number of SYN packets, often with **spoofed source 
   IPs** (tying back to the IP Spoofing concept from Layer 3)
2. The server responds with SYN/ACK to each one and waits for the final ACK
3. Since the source IPs are spoofed/fake, the final ACK never arrives
4. The server's connection table fills up with half-open connections, exhausting 
   memory/resources until it can no longer accept legitimate connections

**Mitigation:**
- **SYN cookies**: instead of storing connection state immediately, the server 
  encodes the necessary state into the SYN/ACK's sequence number itself, so no 
  memory is consumed until the final ACK arrives and proves legitimacy
- **Rate limiting** incoming SYN packets per source IP
- **Firewall/IPS SYN flood protection** features that detect abnormal SYN-to-ACK 
  ratios

---

### 3. Why Port Scanning/Flooding Differs Between TCP and UDP

Because UDP has no handshake, no sequence numbers, and no connection state, 
UDP-based attacks (like UDP floods) rely purely on volume rather than exploiting 
a handshake weakness. TCP-based attacks like SYN flood specifically abuse the 
*stateful* nature of TCP's connection establishment — this is a direct 
consequence of the TCP vs UDP trade-off discussed earlier in this document 
(reliability/state vs. speed/statelessness).

---

### 4. Detection Angle: Well-Known vs Unexpected Ports

From the `netstat -ano` output earlier, we saw well-known ports like `445` (SMB) 
and `135` (RPC) listening locally. From a SOC perspective:

- Unexpected services listening on non-standard ports, or well-known ports 
  (e.g., 443, 22) with traffic that doesn't match the expected protocol, can 
  indicate malware using port camouflage to blend in with legitimate traffic
- Monitoring for **new LISTENING ports appearing** on a host over time is a 
  basic but effective host-based detection technique
