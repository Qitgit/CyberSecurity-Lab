## OSI Layers- 1st Layer Physical Layer
# 1. Overview
The Physical Layer is Layer 1 of the OSI model. It defines how raw bits are transmitted over a physical medium - whether that's copper cable, fiber cable or radio waves(Wi-Fi). This Layer doesn't understand data, IP addresses, or protocols; if it only cares about voltage, signal encoding, and hardware specifications.

# 2. Device Used in this Lab
**Device:** TP-Link Archer VR2100
**Type:** AC2100 wireless MU-MIMO VDSL/ADSL Modem Router
**Ports:** 4X LAN (LAN4 doubles as WAN), 1X VDSL/ADSL port
**Wireless:** Dual-band (2.4GHz / 5GHz)
<img width="480" height="640" alt="IMG_5270" src="https://github.com/user-attachments/assets/ff8565ed-d009-460e-9b89-7645ec81d803" />
<img width="480" height="640" alt="IMG_5271" src="https://github.com/user-attachments/assets/a45d843c-a02a-4f4c-b16c-fde1fd504fd8" />
The images above show the physical ports and antenna setup - this is the actual hardware layer where electrical/radio signals are converted into usable network data.

# 3. Checking Physical Layer Info via CLI
Ran the following command to inspect the wireless connection at the physical layer:
netsh wlan show interfaces
This shows details like signal strength, channel, and transmit/receive rate - values that reflect what's actually happening at layer 1 before any higher-layer protocol (TCP/IP) gets involved.
<img width="684" height="467" alt="netsh wlan show interfaces my wife" src="https://github.com/user-attachments/assets/e307f82e-7041-4c2c-bac7-63b0e0f90817" />

# 4. Why This Matters for Security

Even though layer 1 seems purely physical, it has real security implications:

- *Cable tapping / physical access* : an attacker with physical access to cables can intercept traffic before any encryption at higher layers applies.
- *Rogue access point* : a fake Wi-Fi AP mimicking a legitimate SSID can trick devices into connecting, enabling man-in-the-middle attacks
- *RF jamming* attackers can disrupt wireless signals, causing denial of service
*Practical Context : I've experienced this firsthand at home-running microwave oven often causes Wi-Fi disconnects because both use the 2.4Ghz spectrum. An intentional RF jamming attack uses this exact principle at a higher scale to block communications.

If Layer 1 is compromised, everything built on top of it - such as encryption, authentication, access control - can be bypassed or rendered useless. This is why physical security is just as important as software-based defenses.

# 5. Security Finding : WPA2-Personal Vulnerability

The wireless connection is using **WPA@-Personal** with **CCMP** cipher - which is generally solid, but WPA2 has known, well-documented weakness

### KRACK Attack (key Reinstallation Attack)
Discovered in 2017, this vulnerability exploits the WPA2 4-way handshake process.
An attacker within radio range can force a device to reinstall an already-used encryption key, allowing them to decrypt, replay, or in some cases inject packets into the traffic - without knowing the actual Wi-Fi password.

### Why This Matters
Even with a strong password, WPA2's handshake mechanism itself has a structural flaw. This isn't a misconfiguration issue - it's a protocol-level weakness that affect most WPA2 deployment unless properly patched.

### Remediation
1. **Upgrade to WPA3** where hardware/firmware supports it - WPA3 uses SAE(Simultaneous Authentication of Equals), which fixes the handshake vulnerability that KRACK exploits.
2. **Ensure firmware is patched** - most vendors released patches post-2017 that mitigate KRACK even while staying on WPA2
3. **Use WPA2/WPA3 Transitional mode** if full WPA3 isn't supported by all client devices.
4. **Network segmentation** - even if Wi-Fi is compromised, segmenting IoT/guest devices from critical systems limits blast radius.

### Detection Angle (SOC Relevance)
From a blue team perspective, KRACK-style attacks can be flagged by monitoring for:
- Repeated 4-way handshake retransmissions from the same client
- Unusual deauthentication frame spikes (often used to force handshake retries)
