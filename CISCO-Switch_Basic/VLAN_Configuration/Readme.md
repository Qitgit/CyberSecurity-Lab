# Overview

This lab demonstrates how to configure a VLAN interface (SVI) on a Cisco switch to enable Layer 3 communication between a router, switch, and PC.
By assigning an IP address to VLAN1 and setting a default gateway, the switch becomes reachable and participates in the network.

## Topology

| Device | Interface | IP Address | Purpose |
| --- | --- | --- | --- |
| **Router1 (2811)** | Fa0/1 | 172.17.99.1 | Default gateway for LAN |
| **Multilayer Switch2 (3560)** | VLAN1 | 172.17.99.11 | Switch management interface |
| **PC2** | Fa0 | 172.17.99.21 | Host device |

### Switch VLAN IP configuration

A Layer 2 switch does not have an IP address by default.
To manage the switch remotely (SSH, Telnet, SNMP, Syslog), it must have a reachable IP.

This IP is assigned to a VLAN interface, usually VLAN1:
<img width="1051" height="341" alt="configure switch IP" src="https://github.com/user-attachments/assets/751a1770-f9e6-49cc-baaa-6d6daa465ba8" />

interface vlan1
 ip address 172.17.99.11 255.255.255.0
 no shutdown

This creates a logical Layer 3 interface that allows the switch to act like a host on the network.

### Save Switch Configuration

To ensure the configuration persists after reboot.
Without this, all settings would be lost when the device restarts.

<img width="966" height="141" alt="3To save config" src="https://github.com/user-attachments/assets/8ec624db-3507-4f86-b37c-581bc23ed588" /><br>

*Switch# copy run start*

Saved the running configuration to startup configuration.


### PC Static IP Configuration

<img width="1052" height="456" alt="1configure PC1" src="https://github.com/user-attachments/assets/d2c11f54-c1dd-43a3-bc94-00412e148b28" /><br>

To manually assign a fixed IP for testing connectivity with the switch and router.
Static addressing ensures predictable communication during setup.

### Verify Switch Configuration

Switct# show running-config

To verify connectivity between PC2 and the switch before testing network communication.

<img width="1048" height="359" alt="To check the config" src="https://github.com/user-attachments/assets/225f2945-af62-46a6-af0a-f58afa20dac1" /><br>

Checked interface states on the switch.

### Router Interface Configuration

Configured the router’s FastEthernet0/1 interface.

<img width="1119" height="126" alt="5Router config" src="https://github.com/user-attachments/assets/a29ecba2-6e36-42c4-b1fc-85da32c41b9b" />

This IP (172.17.99.1) serves as the default gateway for the network, enabling routing between subnets.

### Update PC Default Gateway

Set PC2’s default gateway to the router’s IP.

<img width="608" height="181" alt="7PC gateway conf" src="https://github.com/user-attachments/assets/79e2f4ce-4142-4e38-9c5a-43fde916e77e" />

Allows PC2 to send packets beyond its local network via the router.

### Configure Switch Default Gateway

Enables the switch to communicate with devices outside its local VLAN (e.g., router).

<img width="1092" height="119" alt="8switch gateway conf" src="https://github.com/user-attachments/assets/564b1553-6e83-40c1-abe0-5c53aec8aaa0" />

# Summary
Configuring a VLAN on a switch provides network segmentation, security isolation, management access, and routing control.
It is essential for enforcing security boundaries, preventing unauthorized access, and maintaining a clean, well‑structured network.

