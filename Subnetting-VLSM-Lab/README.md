### Overview
 
Large default blocks like `/8`, `/16`, or even `/24` allocate far more host
addresses than most networks actually need — a `/24` gives 254 usable hosts,
but a small department of 10 devices would waste 244 of them. **Subnetting**
solves this by carving a large block into smaller, right-sized pieces, and
**VLSM (Variable Length Subnet Mask)** specifically allows those pieces to be
different sizes depending on how many hosts each one actually needs, instead
of forcing every subnet to be the same size. Beyond address efficiency,
smaller subnets also shrink each broadcast domain (less ARP/DHCP broadcast
traffic per segment) and create logical boundaries where a router can
enforce access control between segments.
 
This activity demonstrates the basic mechanics of VLSM by first confirming
baseline connectivity on a single flat /24 network, then splitting that same
address space into two /25 subnets to show that subnetting alone breaks
connectivity between hosts — even though they're still physically wired to
the same switch. Along the way, a subnet overlap misconfiguration was hit
and resolved, which turned into the most useful part of the exercise.
 
---

---

### Lab / Practical

#### 1. Baseline Topology - /24 Network

Two PCs ('.10' and '.1269') were connected through a single switch on '192.168.56.0/24':

<img width="1059" height="392" alt="Screenshot 2026-08-18 100157" src="https://github.com/user-attachments/assets/aa1e8ee1-4042-42ac-81e8-11cadd26fe65" />

Both hosts were configured with a standard '/24' mask:

<img width="691" height="299" alt="Screenshot 2026-08-18 100248" src="https://github.com/user-attachments/assets/8853da18-8487-4cb0-b121-fecb4105785b" />

pining between the two confirmed normal Layer 2 connectivity within a single broadcast domain:

<img width="641" height="342" alt="Screenshot 2026-08-18 100405" src="https://github.com/user-attachments/assets/ff05973c-cf35-4821-936f-c45e0e89f8be" />

#### 2. Splitting into Two /25 Subnets

The same '/24' block was then split into '192.168.56.0/25' and '192.168.56.128/25', with each PC re-addressed into a different half of the range:

<img width="982" height="309" alt="Screenshot 2026-08-18 100224" src="https://github.com/user-attachments/assets/d4d35469-adf3-4c37-98cb-1f53a8349ffd" />
<img width="698" height="303" alt="Screenshot 2026-08-18 100302" src="https://github.com/user-attachments/assets/2c7e529f-c4fe-42c8-b22f-c982d3cab04e" />
<img width="698" height="291" alt="Screenshot 2026-08-18 100342" src="https://github.com/user-attachments/assets/66fb21a2-9d58-4617-bf71-79ef73afa1e4" />

#### 3. Confirming Subnet Isolation

with no router in place yet, pinging across the two new /25 subnets failed completely

<img width="632" height="286" alt="Screenshot 2026-08-18 100446" src="https://github.com/user-attachments/assets/db0f253e-6a6d-44c0-9b5a-514171638acd" />

**Note:** this was the actual point of the exercise - the physical topology didn't change at all, only the subnet mask did. That single change was enough to put the two PCs into separate broadcast domains, and a Layer 2 switch has no mechanism to route between them. This is a useful hands-on confirmation that subnetting is fundamentally Layer 3 concept, not a cabling one.

#### 4. Overlap Misconfiguration


while bringing up routing between two subnets, an overlap error appeared during interface configuration:
<img width="1233" height="685" alt="Screenshot 2026-08-18 101712" src="https://github.com/user-attachments/assets/9cc1f7d6-bbef-4f59-b0ed-17d77429a0f7" />
<img width="631" height="86" alt="Screenshot 2026-08-18 102106" src="https://github.com/user-attachments/assets/b6308785-5060-41bc-a218-4d8675d0c814" />



**Note:** the error occurred 
because one interface was still holding a '/24' on one interface claims the entire '192.168.56.0-255' range, so Packet Tracer correctly refused to let a second interface claim any part of that same space. 
This was fixed by making sure **both** interface used the correct '/25' mask matching their actual subnet.

#### 5. Fixed Configuration and Verification

After correcting the mask on both interfaces:

<img width="628" height="262" alt="Screenshot 2026-08-18 104751" src="https://github.com/user-attachments/assets/493a2c2f-ff72-43cd-9f78-de5284a0f3ae" />



ping between the two subnets succeeded, now routed through the device instead of failing at Layer 2
<img width="630" height="311" alt="Screenshot 2026-08-18 103952" src="https://github.com/user-attachments/assets/7acd8f33-8545-4ad9-918a-2a82b505eb1f" />

---

### Key Takeaway

The most useful part of this lab wasn't the final working config - it was watching the *same two devices* go from reachable to unreachable purely by changing a subnet mask, with zero changes to physical wiring. It made the abstract idea of "subnetting creates separate broadcast domains" into something directly observable, and the overlap error was a realistic example of how a single leftover misconfigured interface (mask not updated consistently) can block an otherwise correct setup elsewhere in the config.

---

## Security Perspective

### Subnetting as a Segmentation Control

What this lab demonstrates technically - that changing a mask isolates hosts from each other even on the same physical switch - is also the exact mechanism behind network segmentation as a security control.
Splitting a flat network into small subnets limits the blast radius of issues like ARP spoofing, broadcast-based attacks, or a compromised host scanning for lateral movement targets, simply by shrinking the pool of devices that share a broadcast domain with it.

### Overlapping Subnets as a Real-World Risk

The overlap hit in this lab is a toy version of a real misconfiguration risk: overlapping subnet ranges across interfaces, sites, or VPN tunnels can cause routing ambiguity, unintended reachability between segments that were supposed to be isolated, or silent packet drops that are hard to diagnose. 
In an enterprise environment this is exactly the kind of issue that shows up as "why can this device reach a segment it shouldn't be able to," and tracing it back to an addressing plan error is a common part of network/security troubleshooting.
