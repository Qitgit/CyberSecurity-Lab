### Overview

The Presentation Layer is Layer 6 of the OSI model. It's responsible for making sure data is in a format both the sender and receiver can understand - acting as a "translator" between the application layer's data format and the raw bytes that get transmitted across the network.

Key responsibilities of this layer:

-**Translation**: Converting data between different formats (e.g., character encoding differences between systems)
-**Encryption/Decryption**: Securing data before transmission - this is where **TLS/SSL** conceptually sits
-**Compression**: reducing data size before sending, to save bandwidth
-**Serialization**: Converting structured data into a format suitable for transmission

### Why This Layer Is Hard to Isolate

Just like the Session Layer, Presentation Layer responsibilities in real-world networking are absorbed into application protocols rather than existing as a separate, distinct protocol you can filter for in wireshark:

- **TLS**: handles encryption (a Presentation layer job) but is typically discussed as sitting "on the top of" the Transport Layer -this is the layer where the earlier TCP handshake capture's "Client Hello / Server Hello" sequence belongs
- **Character encoding** (UTF-8,ASCII): handled implicitly by applications, rarely something you'd inspect directly in a packet capture
_ **Compression** (gzip, Brotli0): visible in HTTP headers, but the actual compression/decompression happens inside the application, not as a separate wire protocol

Because of the, the clearest way to demonstrate this layer hands-on is by examining a protocol that has **none** of there presentation Layer functions - making the absence obvious by contrast.

---

## Lab / Practical

### Telnet: A Protocol that provides remote access Without Encryption

Captured a Telnet session (port 23) using wireshake to observe that provide no Presentation Layer security fuctions.

<img width="1737" height="274" alt="telnet" src="https://github.com/user-attachments/assets/d77ea9c4-686f-48e5-ad4b-fcb111eb165c" />

The capture shows the Telnet negotiation phase followed by actual session data being exchanged in plain text.

**Key observation:** unlike the TLS-secured protocols, Telnet performs no encryption whatsoever. Every typed during a telnet session - including user name and passwords if authentication occurs over the session - is transmitted as plaintext and fully visible to anyone capturing the traffic.

This makes Telnet a clear negative example for the Presentation Layer's encryption responsibility: where TLS would encrypt data before transmission, Telnet sends it completely unprotected.

---

## Security prospective

### Why Telnet Is Considered Insecure
- **No encryption**: all data, including credentials, travels in plaintext — 
  trivially readable by anyone capturing traffic on the path (a classic case 
  for the "cable tapping" risk discussed in the Physical Layer section)
- **No integrity protection**: data in transit can be modified without detection
- **Legacy protocol**: still found on some legacy network devices (routers, 
  switches, industrial control systems) despite modern alternatives

**Modern replacement:** SSH (Secure Shell) provides the same remote terminal 
access functionality as Telnet, but adds encryption, integrity checking, and 
authentication — fulfilling the Presentation Layer responsibilities Telnet lacks 
entirely.

**Detection (SOC relevance):** any observed Telnet traffic (port 23) on a modern 
network is often itself a red flag — either a misconfigured/legacy device or a 
potential indicator of an attacker using an older, less-monitored protocol for 
access. Many security baselines flag Telnet usage as a finding requiring 
remediation (migrate to SSH).
