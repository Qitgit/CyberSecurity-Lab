### Overview

This activity deploys a **Dedicated DHCP Server**(rather than using the router's built-in DHCP feature, as in Activity 02) to serve LAN B. This mirrors a more enterprise-realistic setup, where DHCP is typically handled by a dedicated server rather than the network's edge router.

---

### Lab / Practical

#### 1. Adding the Server to LAN B

added a 'Server-PT' device (labeled "DHCP") to LAN B, connected to SW2 via FastEthernet0:

<img width="341" height="280" alt="Screenshot 2026-08-12 151016" src="https://github.com/user-attachments/assets/08c97cdd-52d5-4372-af5d-bd918bb1836d" />

#### 2. Initial Static Configuration Conflict

Before configuring the server as a DHCP source, its own management interface was set to a static IP:

<img width="853" height="285" alt="Screenshot 2026-08-12 151358" src="https://github.com/user-attachments/assets/1888b895-7218-4926-a010-6bb224c1d278" />

**Note:** this warning was a useful learning moment — it confirms Packet Tracer 
actively checks for IP conflicts within the same subnet, the same protection 
mechanism discussed in the DHCP CLI activity's `excluded-address` command. The 
address was reassigned to resolve the conflict.

#### 3. Configuring the DHCP Service

Navigated to the server's **Services → DHCP** tab. Initially, the pool 
(`serverPool`) was left with default/empty values — Default Gateway and DNS 
Server both at `0.0.0.0`, and Service left **Off**:

<img width="843" height="467" alt="Screenshot 2026-08-12 151359" src="https://github.com/user-attachments/assets/b1128689-27bf-40d4-a322-d4ee8b6f8faf" />

After configuring the actual address pool for LAN B and enabling the service:

<img width="853" height="461" alt="Screenshot 2026-08-12 152641" src="https://github.com/user-attachments/assets/085a6aa3-1b9a-4c13-bab0-e9dfbe73ec86" />

**Note:** Default Gateway and DNS Server were left at `0.0.0.0` in this pool 
configuration — meaning clients receiving a lease from this server would get an 
IP address but no gateway or DNS information. This is a realistic misconfiguration 
worth documenting rather than hiding, since it affects the client result below.

#### 4. Client Receives a Lease

PC2 was set to DHCP and successfully received an address from the server:

<img width="914" height="278" alt="Screenshot 2026-08-12 152721" src="https://github.com/user-attachments/assets/6307e3f1-a6df-41a0-9b66-18525e6c4735" />

As expected from the pool configuration in Step 3, PC2 obtained a valid IP and 
subnet mask, but **no default gateway or DNS server** — meaning this client 
could communicate within LAN B, but would be unable to reach LAN A (192.168.10.0) 
or resolve any domain names, since neither the gateway nor DNS server fields 
were populated in the DHCP pool.

---

### Key Takeaway

Comparing this activity to Activity 02 (router-based DHCP) highlights a 
practical difference: a dedicated DHCP server gives more granular control 
(multiple pools, service toggles, higher client limits) but also introduces more 
configuration surface area for mistakes — an incomplete pool (missing gateway/DNS) 
silently produces working-but-limited connectivity rather than an outright 
failure, which is a realistic troubleshooting scenario a SOC/network analyst 
might encounter.

---

## Security Perspective 

### DHCP Server Sprawl and Configuration Drift

Running a dedicated DHCP server (in addition to the built-in router DHCP 
capability used in Activity 02) increases the network's configuration surface. 
In a real environment, having multiple potential DHCP sources — even 
legitimate ones — makes it easier to accidentally introduce **two active DHCP 
servers on the same network segment**, causing clients to receive inconsistent 
or conflicting leases depending on which server responds first (the same race 
condition a rogue DHCP attacker would exploit, as discussed in Activity 02).

**Best practice:** only one DHCP server should be authoritative per subnet/VLAN. 
Where redundancy is needed, servers should be explicitly configured to serve 
non-overlapping address ranges (DHCP failover), not left to coincidentally avoid 
collision.

### Incomplete DHCP Pools as an Availability Risk

The missing gateway/DNS values in this lab, while accidental, illustrate a real 
operational risk: a misconfigured DHCP pool can silently degrade an entire 
subnet's connectivity without triggering an obvious error — clients appear to 
have a working IP and don't immediately signal "broken" the way a failed DHCP 
request does. This is a useful category of issue for a SOC/NOC analyst to be 
aware of when triaging "some users can't reach X" tickets that turn out to be 
DHCP scope issues rather than security incidents.
