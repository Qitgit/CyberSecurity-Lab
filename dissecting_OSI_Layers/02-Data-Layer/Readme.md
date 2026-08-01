### Overview

The Data Link layer is 2 of the OSI model. It defines how devices on the same local network (same physical or logical segment) communicate with each other using hardware addresses rather than IP addresses. While the Physical Layer only cares about raw signal transmission, the Data Link Layer adds structure - it organizes bits into **frames**,handles error detection, and manages access to the shared medium.

Key responsibilities of this layer :

-**Framing**: wrapping raw bits into structured units called frames, which include source/destination MAC addresses and error-checking data(FCS -Frame check Sequence)
-**MAC Addressing**: uniquely identifying devices on the local network using a hardware-burned address
**Error Detection**: using checksums to detect (not correct) transmission errors
-**Media Access Control** determining how devices share access to the same physical medium without constantly colliding(e.g., CSMA/CD in older Ethernet)

The Data Link layer is split into two sublayers:
-**LLC (Logical Link Control)**: handles flow control and error checking
-**MAC (Media Access Control)**: handles addressing and channel access

This layer operates independently of what's happening at Layer 3 (IP) and above - it doesn't know or care about the final destination across the internet, only about "who is my direct neighbor on this network segment."

---

### MAC address structure
FF:FF:FF:FF:FF:FF
-First 3 bytes(24bits) = **OUI**,identifying the manufacturer (assigned by IEEE)
-Last 3 bytes = a unique serial number assigned by that manufacturer
-In theory globally unique, but can be spoofed via software - this becomes a security concern discussed below

###How Switches work

Unlike a hub (which blindly repeats signals to every port - inefficient and easy to eavesdrop on), a switch is smarter.
-Learns a **MAC address table (CAM table)**:  which MAC address lives on which port
-When a frame arrives, it forwards it to the port matching the destination MAC
-If the destination MAC is unknown, it flood the frame to all ports, then learns the mapping once a reply comes back
-**Broadcast**frames ('FF:FF:FF:FF:FF:FF)are always sent to all ports

---

### ARP(Address Resolution protocol)

Used when a device knows the target's IP address not its MAC address.

**Flow**
1. Host 'A' knows Host 'B''s IP, but not its MAC
2. a sends an ARP Request as a **broadcast**("Who has 192.168.0.5?") - destination MAC = 'FF:FF:FF:FF:FF:FF'
3. Only the host with that IP(B) responds with an ARP Reply via **unicast**including its MAC address
4. A caches this IP-MAC mapping in its ARP table to avoid repeating the lookup

### ARP is LAN-Scoped Only

ARP Requests are sent as **Layer 2 broadcasts**, and routers do not forward broadcast traffic between networks by design - otherwise broadcast storms would cripple the internet. This means ARP only functions within a single local network segment (same subnet).

When communicating with a host outside the local network:
1. The sending device checks if the destination IP is outside its own subnet
2. If it is, the device ARPs for its **default gateway's MAC address** instead of the final destination's MAC
3. The frame is sent with the destination IP unchanged (final target), but the destination MMAC set to the gateway/router
4. At each hop, the router rewrites the Layer 2 (MAC) header while keeping the Layer 3 (IP) header intact - MAC addresses change hop-by-hop, but IP addresses stay the same end-to-end

**Security implication**: This is exactly why ARP spoofing attacks are limited to attackers who are already inside the same local network - ARP traffic never leaves the LAN, so an external attacker has no way to inject or observe it.

---

###VLAN (Virtual LAN) - Reference

Even when devices are physically connected to the same switch, VLANs allow logically separating them into different broadcast domains. This is done by tagging Ethernet frames (802.1Q) Commonly used in enterprise networks to isolate traffic by department or function.

---
## Security Perspective

### 1. MAC Spoofing
Software can change a device's reported MAC address to impersonate another device.
Used to bypass MAC filtering or hijack another user's identity on the network.

### 2. ARP Spoofing / ARP Poisoning
An attacker sends forged ARP Replies to corrupt a victim's ARP cache
-For example
claiming " I am the gateway," causing the victim's traffic to route through the attacker (Man-in-the-Middle). This works because ARP has **no authentication** - any reply is trusted by trusted by default, which is a fundamental design weakness of Layer 2.

##Detection approach**
-Multiple MAC addresses observed for the same IP within a short time window is a red flag
-Switch-level protections like Dynamic ARP Inspection (DAI) can block this 
-wireshark may flag "Duplicate IP address configured" warnings as a hint

### 3. MAC Flooding
A switch's CAM table has limited capacity. An attacker can flood it with fake MAC addresses, forcing the switch to fail open and broadcast all traffic like a hub — 
making eavesdropping trivial.

### 4. VLAN Hopping
An attack technique where crafted frames are used to break out of one VLAN and access traffic on another VLAN that should be isolated.

---

## Lab / Practical

*(Screenshots and analysis to be added here: `ipconfig /all`, `ping`/`tracert` 
to 8.8.8.8, and Wireshark ARP capture with MAC/OUI lookup)*
**ipconfig /all**
<img width="778" height="518" alt="ipconfig" src="https://github.com/user-attachments/assets/df2f2255-7c17-4b43-8c17-a86163d777fa" />
## MAC Address with OUI
`9C-C7-D3-FF-FF-FF` is a MAC address. The first **3 bytes** (`9C-C7-D3`) represent 
the OUI (Organizationally Unique Identifier), which identifies the manufacturer. 
The remaining 3 bytes (`FF-FF-FF`) are the device-specific serial number assigned 
by that manufacturer.

**Wireshark_ARP**
<img width="991" height="148" alt="Wireshark_ARP" src="https://github.com/user-attachments/assets/a09aa2d0-91cf-4bac-a3d9-92a49cf1f174" />

**Ethernet II**
<img width="903" height="128" alt="Ethernet 2" src="https://github.com/user-attachments/assets/4f7b55e2-84fe-4259-9f22-a05657885f10" />
### Note on LLC

The LLC (Logical Link Control) sublayer is not present in this capture. This is 
because the frame uses the **Ethernet II** format, which identifies the upper-layer 
protocol directly via the EtherType field (`0x0806` = ARP). LLC headers only appear 
in **IEEE 802.3 raw** frames, which use a Length field instead of Type and therefore 
need an LLC/SNAP header to indicate the upper-layer protocol. Since virtually all 
modern networks use Ethernet II, LLC headers are rarely seen in practice.

**ARP Request**
<img width="880" height="169" alt="ARP request" src="https://github.com/user-attachments/assets/c2ab5d2f-e1ec-40c7-86fb-2b2aa87f0e8f" />



