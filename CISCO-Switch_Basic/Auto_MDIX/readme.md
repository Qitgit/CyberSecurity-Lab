# overview
**Auto-MDIX** os an Ethernet freature that *automatically detects whether a straight-through or crossover cable is needed* and internally swaps the transmit/receives pairs so the link comes up correctly. In modern networks, this is why you can plug almost any Ethernet cable between almost any two devices and the connection "just works." 

## Enabling Auto-MDIX on Cisco Switches
This lab demonstrates how different cable types behave when connecting two Cisco Catalyst 3560-24PS switches, and how enabling Auto‑MDIX allows the switches to automatically adjust for straight‑through or crossover cabling.

*straight-through*
When Auto‑MDIX is not enabled, connecting two switches with a straight-through cable results in a link failure.
<img width="848" height="323" alt="MDIX wrong cable conn" src="https://github.com/user-attachments/assets/6c2d3e21-a33a-4bcf-a58a-f49a69466e88" />
The image shows SW1 and SW2 connected via Fa0/1 using a straight cable, but the link does not come up.

*Crossover cable*
Using a crossover cable, the switches successfully establish a link even without Auto‑MDIX.
<img width="848" height="323" alt="MDIX wrong cable conn" src="https://github.com/user-attachments/assets/2aa47dcf-a036-47fe-b924-698ca6867f31" />

This image shows SW1 and SW2 connected with a crossover cable, and the link is operational.

*Configure the auto MDIX*
Auto‑MDIX allows the switch to automatically detect cable type and adjust the interface, making both straight‑through and crossover cables work interchangeably.
<img width="1088" height="364" alt="config mdix" src="https://github.com/user-attachments/assets/cade5605-4f95-4e8e-97b8-39c1c2324904" />

Switch> enable
Switch# configure terminal
Switch(config)# interface FastEthernet0/1
Switch(config-if)# mdix auto

## Security Relevance
Although Auto-MDIX is a Layer 1 feature, it plays an important role in in network security.  
It improves availability, reduces physical misconfiguration risks, and ensures reliable connectivity during incident response or forensic analysis.  
This contributes directly to secure network operations and supports the availability pillar of the CIA Triad.
