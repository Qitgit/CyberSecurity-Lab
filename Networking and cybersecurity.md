# Networking and Cybersecurity Fundementals (20/02/26)

### 1. Key concepts
* **IP (Internet protocol)** : A dynamic digital address for identifying a device on a network.
* **MAC (Media access control)** : A Permenent hardware ID assigned to a network interface.
* **Default gate** : The "exit door"(Router) used to communicate with external networks.
* **ARP  (Address resoloution protocol)** : A protocol used to map an IP address to a physical MAC address.
* **DHCP (Dynamic Host Configuration Protocol)** : A service that automatically assigns IP addresses to devices.

### 2. Commands I used today

* ip addr : to check my ip addess
* nmcli device show : it has shown me the default gate, IP address and MAC address.
*arp -a : to discover the arp cache

### 3. Security Lessons Learned
* **Spoofing** : Understanding that MAC and ARP entrie can be faked is critical for defense.
* **Conectivity** : Use 'Ping' to verify if a path exists between two IP addresses.
