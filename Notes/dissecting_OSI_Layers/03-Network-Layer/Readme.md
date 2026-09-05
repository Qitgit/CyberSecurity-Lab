### Overview

The Network Layer is 3 of OSI model. While Layer 2 handles communication between devices on the *same* local network using MAC address, Layer 3 is responsible for getting data from one network to another - potentially across the entire internet - using **IP addresses**.

Key responsibilities of this layer

-**Logical Addressing**: assigning IP addresses that identify a device's network and host portion (e.g., '192.168.1.999'
-**Routing**: determining the best path for a packet to travel across multiple networks, hop by hop, via routers
-**Packet Forwarding**: routers inspect the destination IP and decide which interface/next-hop to forward the packet to
-**Fragmentation**: breaking packets into smeller pieces if they exceed the MTU(Maximum Transmission Unit) of a given link.

Unlike MAC addresses, IP addresses are **hierarchical** -structured into network and host portions- which is what makes routing at scale possible.
A router doesn't need to know every single host in the world; it just needs to know which direction to send traffic for a given network range.

Key protocols at this layer: **IP (IPv4/IPv6)**, **ICMP**, **routing protocols**

##Lab / Practical

### Tracert / TTL Analysis

Ran 'tracert 8.8.8.8' and captured the underlying ICMP traffic with wireshark

<img width="1002" height="345" alt="tracert" src="https://github.com/user-attachments/assets/f5bce439-c238-4c30-8920-d41c9fda6a5f" />


**How traceroute works**
1. The sender transmits an ICMP Echo  Request with **TTL = 1**
2. The first router decrements TTL to 0, drops the packet, and replies with "Time-to-live exceeded" - revealing hop 1's IP
3. The sender resends with **TTL =2**, and the process repeats - hop 2 replies this time, since hop 1 now forwards it successfully
4. This continues, incrementing TTL by 1 each round, until the destination (8.8.8.8) is finally reached


<img width="1077" height="356" alt="wireshark-ICMP" src="https://github.com/user-attachments/assets/9ecef62f-d789-495d-945a-c8ea5baf143a" />


This is visible directly in the packet list - TTL=4 packets receive "Time-to-live exceeded" from "172.31.21.9' (hop 4), TTL=5 from '116.225.26.68' (hop 5), matching the 'tracert' output line by line.

**Note on hop 4:** '172.31.21.9' falls within the private range '172.16.0.0/12'.
This suggests the ISP uses internal (carrier-grade NAT or internal routing) addressing for part of its infrastructure before traffic reaches public hops.


##Security perspective

### 1. ICMP-based Reconnaissance

Tools like 'ping' and tracert'. which were used in this lab for legitimate network troubleshooting, can also serve as reconnaissance tools for an attacker.

- **Ping sweep**: sending ICMP Echo Requests across an entire subnet to identify which hosts are alive

- -**Traceroute Reconnaissance**: mapping out network topology - router locations, hop count, and potentially firewall/gateway placement - before planning further attacks.

The 'tracert 8.8.8.8' capture in this lab illustrates exactly what an attacker's tool output would look like : each hop's IP, response time, and path structure are all revealed through ICMP TTL-expiry responses.

**Detection**: a sudden spike of ICMP Echo Requests from a single source targeting an entire IP range in a short time window is a strong indicator of active scanning and should be flagged by an IDS/SIEM correlation rule.

---

### 2. TTL-Based OS Fingerprinting

Different operating systems use different default TTL values:

Window - 128
Linux/Unix - 64
Some network devices - 255

Since TTL decreases by 1 at each hop, an attacker receiving a packet can estimate  the original TTL and reverse-engineer the sender's likely OS. For example, a received TTL of 118 (128 - 10 hops) suggests the source is window host.

** Security relevance**: this same logic works both ways - a blue team analyst can use unexpected TTL values as an anomaly signal. If traffic claiming to originate from a known Linux server arrives with a TTL pattern consistent with windows, that's a red flag worth investigating (possible spoofing or unauthorized device).

### 3. Smurf Attack (ICMP + IP Spoofing + Broadcast)

A classic DDoS technique that combines IP spoofing with ICMP:

1. The attacker sends an ICMP Echo Request to a network's **broadcast address**
2. The **Source IP is  spoofed** to be the victim's IP address, not the attacker's own.
3. Every host on that network receives the broadcast and replies - but all replies are sent to the (spoofed) victim IP instead of the attacker
4.The victim gets flooded with a massive number of ICMP replies from many hosts at once - a traffic amplification DDoS attack

 This attack works specifically because ICMP has**no source verification**- combined with IP spoofing (Layer 3 has no built-in authentication of source addresses either) and broadcast forwarding, it becomes a powerful amplification vector.

 **Mitigation**:
 -Disable IP-directed broadcasts on router ('no ip directed-broadcast')
 -Implement **egress/ingress filtering** (BCP 38) to block packets with spoofed source IPs from leaving or entering a network
 -Rate-limit ICMP traffic at the network edge
