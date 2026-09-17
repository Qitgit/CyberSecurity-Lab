# VLAN Trunking & Native VLAN security Lab

Cisco Packet Tracer lab covering 802.1Q trunking between two switches, a live DTP-based trunk negotiation vulnerability,  and a native VLAN segmentation bypass discovered by physically moving a interface cable between ports. 

---

## 1. What Trunking Acutually Is


Trunking is a single physical link that carries traffic for **multiple VLANs simultaneously**, using 802.1Q tags to identify which VLAN each frame belongs to. Two swithces connected by a plain access link cannot separate VLAN10 from VLAN20 traffic - trunking is what makes that seperation possible over one shared cable.

### Topology

- Switch3: PC1 (VLAN10, Fa0/10), PC3 (VLAN20, Fa0/20)
- Switch4: PC2 (VLAN10, Fa0/10)
- Uplink: Switch3 Fa0/1 — Switch4 Fa0/2
- Subnet: 192.168.1.0/24

<img width="1142" height="542" alt="1_Topology of Trunking" src="https://github.com/user-attachments/assets/b425f26f-d2a5-4dc7-9c6d-749775603896" />

---

## 2. Static Trunk Configuration


```
Switch3#conf t
Switch3(config)#int f0/1
Switch3(config-if)#switchport mode trunk
```

<img width="1334" height="473" alt="2_setting trunk port" src="https://github.com/user-attachments/assets/7202dc72-4395-496c-b9d3-5820cf740afb" />

'show int trunk' confirms the result: 802.1Q encapsulation, status trunking, VLANs 1, 10, 20 active.

<img width="1362" height="481" alt="3_Verifiying trunk" src="https://github.com/user-attachments/assets/8b41d17e-1d88-4d89-ae41-0d2e51bcea56" />

---

## 3. DTP Negotiation - An Unintended Vulnerability

Switch4's Fa0/2 was never explicitly configured as a trunk. It was left at its default dynamic state.`show int trunk` on Switch4 shows
encapsulation as `n-802.1q` (**n** = negotiated) and Administrative
Mode `auto` — meaning Switch4 only became a trunk because Switch3 sent
**DTP (Dynamic Trunking Protocol)** frames requesting it.

 <img width="1060" height="600" alt="4_Verifiying DTP protocol" src="https://github.com/user-attachments/assets/f67468e7-7668-4a10-8806-9bf9634e2883" />
 
**Why this matters:** DTP is enabled by default on most access ports
(`dynamic auto` / `dynamic desirable`). This is the exact mechanism
behind a **DTP spoofing / VLAN hopping attack**:
 
1. An attacker plugs a laptop into any access port still running DTP.
2. A tool such as Yersinia sends a forged DTP Desirable frame.
3. The switch interprets this as a legitimate trunk request and
   converts the port to a real trunk.
4. The attacker's NIC now receives every VLAN's tagged traffic on that
   link and can inject frames into any VLAN via 802.1Q subinterfaces.
No firewall or ACL stops this — it exploits the fact that trunk
formation was allowed to be negotiated at all, rather than fixed by an
administrator.
 
**Mitigation** (not applied in this build, documented as the correct
fix):
 
```
! Every access port
switchport mode access
switchport nonegotiate
 
! Every intentional trunk port
switchport mode trunk
switchport nonegotiate
```
 
`nonegotiate` disables DTP entirely on that port, so trunk status can
never change without an explicit configuration command.
 
---
 
## 4. VLAN Isolation Verification
 
- PC1 (VLAN10) → PC2 (VLAN10): success, 0% loss.
- PC3 (VLAN20) → PC2 (VLAN10): 100% loss.
No router or Layer 3 SVI exists in this topology, so VLANs remain
fully isolated broadcast domains — this failure is expected, not a
fault.

<img width="1338" height="498" alt="5_Verifiying Vlan" src="https://github.com/user-attachments/assets/9cf5c755-15ad-439d-ace9-b7602b3e3035" />
<img width="1385" height="517" alt="6_Verifying Vlan" src="https://github.com/user-attachments/assets/c3098b2c-47f4-4561-aad5-5e6b07f54df5" />
<img width="1047" height="712" alt="7_Ping between Vlan 10" src="https://github.com/user-attachments/assets/fc132163-51a9-4849-9c3a-a7b23c1d3c63" />
<img width="1045" height="825" alt="8_Ping between Vlan 20 to 10" src="https://github.com/user-attachments/assets/46143857-b37f-4b24-921a-1ace3daff363" />

 
---
 
## 5. Orphaned DTP Trunk + Native VLAN Bypass
 
### Setup
 
Switch3 was physically disconnected from Switch4 **after** their trunk
had already been dynamically negotiated via DTP. Fa0/2's Administrative
Mode remained `dynamic auto`, but its Operational Mode stayed **trunk**
— the port did not revert to access just because its neighbor
disappeared.
 
PC3 — previously an access-mode host on Switch3 labeled VLAN20 — was
then plugged directly into Fa0/2, the now-orphaned trunk port. A second
host, PC0, was connected to Fa0/24, an unconfigured access port
defaulting to VLAN1.
 
 <img width="1047" height="666" alt="9_Native Vlan" src="https://github.com/user-attachments/assets/29d62cd4-3610-423f-aa82-b3830e1e8650" />
 
### Result — Before Any Native VLAN Change
 
PC3 successfully pinged PC0 with 0% loss, despite carrying its prior
"VLAN20" label.

 <img width="1043" height="620" alt="10_Ping to New PC" src="https://github.com/user-attachments/assets/0cf18c50-e821-4ca5-bc73-0b7e642131aa" />
 
**Why this happens:** PC3 sends plain untagged Ethernet frames — a PC
has no concept of 802.1Q tags or VLAN membership. When those untagged
frames arrive on Fa0/2, which is operating as a trunk port, 802.1Q
rules classify any untagged frame as belonging to that port's
**native VLAN**. Fa0/2's native VLAN was still the default (VLAN1) at
this point, and PC0 (Fa0/24, also defaulting to VLAN1) sits in the same
VLAN. Both hosts land in VLAN1 — PC3's prior VLAN20 label had no effect
on the outcome.
 
This is a **VLAN segmentation bypass that requires zero attacker
tooling**: a leftover, dynamically-negotiated trunk port silently
re-admits whatever device is plugged into it into the native VLAN,
overriding whatever VLAN that device was administratively "supposed"
to belong to.
 
### Fix Applied
 
```
Switch4(config)#vlan 666
Switch4(config-vlan)#name BAD-DATA
Switch4(config-vlan)#exit
Switch4(config)#int f0/2
Switch4(config-if)#switchport mode trunk
Switch4(config-if)#switchport trunk native vlan 666
```

 <img width="1049" height="265" alt="11_Changing default vlan" src="https://github.com/user-attachments/assets/44e281e6-aa89-405b-97da-01e1a8709281" />
 
`show int trunk` confirms Fa0/2's native VLAN is now 666.

<img width="1038" height="358" alt="12_Verifying native vlan change" src="https://github.com/user-attachments/assets/e3378b86-a8ae-40b4-b788-a166febe6d4f" />

### Result — After the Change
 
The same PC3-to-PC0 ping now fails (100% loss). PC3's untagged frames
are classified as VLAN666 on ingress and no longer match PC0's VLAN1 —
the accidental VLAN1 bypass is closed.
 
<img width="1049" height="551" alt="13_PIng 20 to New PC to Verifying Native Vlan works" src="https://github.com/user-attachments/assets/7a5853df-2ac1-43b7-b503-721ef3b2eb18" />

### Root Cause and Correct Hardening
 
The root cause was never simply "native VLAN = 1." It was that
**Fa0/2 remained a dynamically-negotiated trunk port after its intended
trunk neighbor disappeared**, so a plain access-mode PC plugged into it
inherited trunk-port native-VLAN behavior instead of being isolated as
an access host. A complete fix addresses both layers:
 
```
! If Fa0/2 should serve access hosts going forward
Switch4(config)#int f0/2
Switch4(config-if)#switchport mode access
Switch4(config-if)#switchport nonegotiate
 
! If a trunk is still genuinely required at that port
Switch4(config-if)#switchport mode trunk
Switch4(config-if)#switchport nonegotiate
Switch4(config-if)#switchport trunk native vlan 666
```
 
`nonegotiate` removes the dependency on DTP state persisting correctly
after a link change. A non-default native VLAN removes VLAN1 as a
silent fallback for any untagged host — but only if VLAN666 (or
whichever VLAN is chosen) is never assigned as an access VLAN on a real
host, or that host becomes the new bypass point instead.
 
---
 
## SOC Analyst Relevance
 
- A port whose **Administrative Mode** (`dynamic auto`) no longer
  matches its **Operational Mode** (`trunk`) after a topology change is
  a concrete detection signal. `show interfaces switchport` across
  access-layer ports should be part of a periodic configuration audit,
  not something checked only during incident response.
- A native VLAN mismatch between trunk peers triggers
  `%CDP-4-NATIVE_VLAN_MISMATCH` — worth alerting on directly.
- This lab shows that VLAN membership enforcement depends on **current
  port configuration state**, not on a device's prior identity or
  label — directly relevant when investigating whether a host that
  moved between switch ports retained its intended network segment.
- DTP-based trunk formation on a port that should be access-only is
  itself a finding worth flagging in a configuration review, independent
  of whether it's ever actively exploited.
<img width="1142" height="542" alt="1_Topology of Trunking" src="https://github.com/user-attachments/assets/47fc0c41-eb0d-430d-aedd-b8588468e1c2" />

