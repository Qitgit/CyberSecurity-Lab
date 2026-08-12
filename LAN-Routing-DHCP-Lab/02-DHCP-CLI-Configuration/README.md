### overview

This activity configures **RT1** as a DHCP server for LAN A using IOS CLI commands, replacing the static IP addressing used in the previous activity.
The goal is to demonstrate automate IP address assignment and observe the full request-to-lease process from the both router and client perspective.

---

### Lab / Practical

#### 1. Before configuration : DHCP Request fails

With PC1 set to DHCP but no DHCP server yet configured on the network, the request times out:

<img width="874" height="700" alt="Screenshot 2026-08-12 142145" src="https://github.com/user-attachments/assets/76fad7d7-e8e0-4d51-9588-b67db2e70c41" />

This confirms DHCP is entirely dependent on a server being reachable - without one, a client has no way to obtain an IP address and falls back to no connectivity (or APIPA in a real windows environment).

#### 2. Configuring RT1 as a DHCP Server

Entered global configuration mode:

<img width="172" height="43" alt="Screenshot 2026-08-12 150319" src="https://github.com/user-attachments/assets/d6b3ff33-64f9-4170-a3be-1bec5d72f02d" />

Excluded the router's own gateway address (and reserved addresses up to '.10')from the DHCP pool, so the server never hands these out to clients:

<img width="767" height="40" alt="Screenshot 2026-08-12 150331" src="https://github.com/user-attachments/assets/48f03229-ca84-443b-bc1a-1a4a59d33136" />

Created the DHCP pool for LAN A, defining the network range, default gateway, and  DNS server to be handed out to clients:

<img width="628" height="82" alt="Screenshot 2026-08-12 150408" src="https://github.com/user-attachments/assets/ac62f4e3-3598-4d55-b714-cf40f7924409" />

Re-confirmed the router's own interface IP(unrelated to the DHCP pool it self, but required for the interface to be active) and brought the interface up:

<img width="578" height="109" alt="Screenshot 2026-08-12 150416" src="https://github.com/user-attachments/assets/13924a99-89c1-4349-b7a0-ce6a04cc0a22" />

#### 3. After Configuration: DHCP Request Succeeds

<img width="856" height="298" alt="Screenshot 2026-08-12 150458" src="https://github.com/user-attachments/assets/afc48ef4-0a8d-413a-9819-a1498564c4ba" />

PC1 received '192.168.10.11' - the first address available after the excluded range ('192.168.10.1'-'10'), confirming the exclusion range was applied correctly by the DHCP pool.

---

### Key Takeaway

This demonstrates DHCP entirely from the **Network infrastructure side** - the router itself acts as the DHCP server, a common setup on small/home networks 
where a dedicated DHCP server isn't available. The `ip dhcp excluded-address` 
command is a practical safeguard: without it, the DHCP pool could hand out the 
router's own gateway IP to a client, causing an IP conflict.

---

## Security Perspective 

Because DHCP requests are broadcast (`DHCPDISCOVER`) and the client accepts the 
**first valid offer it receives**, an attacker running an unauthorized DHCP 
server on the same network can hand out malicious configuration — most notably, 
setting themselves as the default gateway or DNS server, enabling a Man-in-the-
Middle position over all of that client's traffic.

**Mitigation:**
- **DHCP Snooping**: a switch-level feature that only allows DHCP server 
  responses (`DHCPOFFER`, `DHCPACK`) from designated trusted ports — any rogue 
  DHCP server plugged into an untrusted port gets its responses dropped
- Monitoring for unexpected `DHCPOFFER` messages originating from unauthorized 
  MAC addresses/ports on the network
