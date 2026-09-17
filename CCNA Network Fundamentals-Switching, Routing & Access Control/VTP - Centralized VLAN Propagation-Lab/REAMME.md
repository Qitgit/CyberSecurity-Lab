# VTP (VLAN Trunking Protocol) — Centralized VLAN Propagation Lab
 
## Objective
Configure a VTP Server/Client topology to centrally manage VLAN databases across
multiple switches, verify that VLAN changes propagate automatically to clients,
confirm that clients are blocked from making local VLAN changes, and analyze the
security implications of VTP's trust model.
 
**Why this matters:** In a network with many switches, manually creating/renaming
VLANs on every device is error-prone and hard to keep consistent. VTP centralizes
that in one Server switch and propagates the VLAN database to Client switches over
existing trunk links. It does **not** create or manage trunks — trunking is
configured independently on every switch.
 
## Topology
Client1 (Fa0/1) — Server (Fa0/1, Fa0/2) — Client2 (Fa0/2)
 
All three switches are 2960-24TT. Trunk links established manually on Fa0/1–2
of the Server before any VTP configuration (`switch mode trunk`).

 <img width="752" height="308" alt="1 VTP Topology" src="https://github.com/user-attachments/assets/53181313-7db1-4735-89bc-a58be7188a48" />
 
## Configuration
 
### 1. Establish trunks (prerequisite — not part of VTP itself)

<img width="1663" height="374" alt="2 DTP" src="https://github.com/user-attachments/assets/a752b435-11a3-4688-af30-a4ad0e444ab0" />


```
Switch(config)#int range f0/1 - 2
Switch(config-if-range)#switch mode trunk
```
 
### 2. Configure VTP Server

<img width="1482" height="204" alt="3 assign the VTP Server" src="https://github.com/user-attachments/assets/e7cdcbef-8b32-4c1b-95ba-6d456e75a7c5" />


```
Switch(config)#vtp domain smtafe.wa.edu.au
Switch(config)#vtp password P@ssw0rd
```
Default mode on a 2960 is Server, so no explicit `vtp mode server` was required.
 
### 3. Configure VTP Clients

<img width="1272" height="348" alt="4 assign the VTP Client" src="https://github.com/user-attachments/assets/38f460ae-bb89-41d3-bb40-ee3614a8f022" />


```
Switch(config)#vtp mode client
Switch(config)#vtp domain smtafe.wa.edu.au
Switch(config)#vtp password P@ssw0rd
```
**Observation:** before the domain was typed manually, the client already reported
`Domain name already set to smtafe.wa.edu.au` — the domain name propagated
automatically the moment the trunk link came up, via VTP summary advertisements.
The password is *not* carried in these advertisements (only an MD5 digest of the
VLAN database + password is used for authentication), so it had to be entered
manually on the client to match the server.
 
### 4. Verify baseline state (before any VLAN change)
Server:
```
VTP Operating Mode      : Server
Configuration Revision  : 0
Number of existing VLANs: 5
```
Client1: identical domain, revision 0, mode Client. Confirms both switches are in
the same VTP domain before any VLAN data has been pushed.
 
<img width="1504" height="376" alt="5 Verifying server VTP" src="https://github.com/user-attachments/assets/133c9331-5935-468d-ba1c-fca06327d149" />
<img width="1297" height="329" alt="6 Verifying client VTP" src="https://github.com/user-attachments/assets/8368fc41-981e-4c68-88a7-643c44f08468" />

 
### 5. Push a VLAN change from the Server

<img width="878" height="223" alt="7 Give changes to VTP server" src="https://github.com/user-attachments/assets/53d2cd00-8581-4078-b096-31068673cbdc" />


```
Switch(config)#vlan 10
Switch(config-vlan)#name A
Switch(config-vlan)#vlan 20
Switch(config-vlan)#name B
Switch(config-vlan)#vlan 30
Switch(config-vlan)#name C
Switch(config-vlan)#vlan 40
Switch(config-vlan)#name D
```
Result on Server: Configuration Revision jumps **0 → 8** (4 VLAN creations + 4
renames = 8 database writes) and existing VLANs go from 5 → 9.
 
<img width="1261" height="351" alt="8 Verify the changes on server VTP" src="https://github.com/user-attachments/assets/5c01e0f3-9392-4770-a9b2-696afb7ed35d" />

 
### 6. Confirm propagation to Client
Client1 `show vtp status` immediately reflects Revision 8, 9 VLANs — pulled from
the Server without any manual configuration on the client.
 
<img width="1122" height="323" alt="9 Verify the changes on client VTP" src="https://github.com/user-attachments/assets/b4fc2fa1-0d20-4505-844c-871b83d4cf5b" />

 
### 7. Confirm Client cannot make local VLAN changes

<img width="966" height="139" alt="10 attempt to update the client VTP" src="https://github.com/user-attachments/assets/1d442967-8994-4b08-89a0-4f40951b81bf" />


```
Switch(config)#no vlan 10
% VTP VLAN configuration not allowed when device is in CLIENT mode.
```
This confirms the trust boundary: Client mode is read-only for VLAN data by
design — the VLAN database can only be authored on the Server.
 

 
## Security analysis (why this matters for SOC work)
 
VTP's propagation model is a **trust-by-domain-and-password** system with no
strong authentication in VTPv1/v2, and it has a well-known failure mode:
 
- **VLAN database overwrite ("VTP bombing")**: any switch that joins the domain
  with a matching domain name/password and a *higher* configuration revision
  number than the current Server will have its VLAN database accepted as
  authoritative by every other switch in the domain — including a Server. A
  rogue or misconfigured switch (e.g. one previously used in a lab, still
  carrying a high revision counter, plugged into a live trunk) can silently
  wipe production VLANs across the whole domain. This lab reproduced the
  revision-increment behavior (step 5) that makes this attack possible.
  
- **Password confidentiality**: the VTP password itself is never transmitted;
  advertisements only carry an MD5 digest computed from the password and VLAN
  database. This resists passive password disclosure but is still vulnerable
  to offline attack if the digest is captured, since MD5 is not
  collision-resistant against modern hardware.
  
**Mitigations:**
  
- Set switches that don't need to receive VLAN pushes to `vtp mode transparent`
  rather than Client, so they can't be forced to accept a rogue database.
  
- Reset configuration revision to 0 (`vtp mode transparent` then back, or change
  domain name and back) on any switch before reintroducing it to a production
  trunk.
  
- Prefer VTP version 3, which supports primary/secondary server authentication
  and closes the "any higher revision wins" weakness of v1/v2.
- Where VLAN counts are small and static, disable VTP entirely and manage VLANs
  per-switch — smaller blast radius than a shared propagation domain.
  
## Key takeaways

- VTP centralizes VLAN database management.
- Domain name propagates over an established trunk before any manual VTP
  config is entered; password does not.
- Client mode enforces read-only VLAN state locally — verified via a failed
  `no vlan` attempt.
- VTP's revision-based trust model is also its main attack surface.
