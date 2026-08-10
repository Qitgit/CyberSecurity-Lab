### Overview

The Application Layer is Layer 7 of the OSI model - the topmost layer, and the only one that directly interacts with the end user or user-facing software.
While Layers 1-6 purely to support getting data from one point to another reliably and in a usable format, Layer 7 is where that data is actually put to use.

Key responsibilities of this layer:

- **Providing network services to applications**: Web browsing (HTTP/HTTPS), email (SMTP, IMAP), file transfer (FTP), remote access (SSH, Telnet -covered in the Presentation Layer section), and name resolution(**DNS**)
- **User-facing protocols**: unlike lower layers, application layer protocols are the ones a person (or their software) directly initiates a request through
- **Data formatting for the specific service**: each protocols defines its own message structure (e.g., an HTTP request, an SMTP email, a DNS query)

- DNS is one of the clearest examples of an Application Layer protocol - it's a complete, self-contained service that a huge number of other applications (browsers, email clients, virtually everything_ rely on before they can even start communicating.

- ---

## Lab / Pracical

### 1. Basic DNS Query - Resolving a Domain with Aliases

Ran 'nslookup www.cisco.com' to resolve a domain name to its IP address 

*<img width="386" height="247" alt="Nslookup" src="https://github.com/user-attachments/assets/3c96e557-e132-4cf5-a9fd-36659a3cc0ab" />*

Notice the response includes **both IPv4 and IPv6 addresses**, along with a chain of aliases -'www.cisco.com' doesn't resolve directly to an IP; it points to another name, which points to another, before finally resolving.

### 2. Observing the CNAME Chain in Wireshark

Captured the actual DNS query/response packets behind the 'nslookup' above

*<img width="1700" height="36" alt="Cname" src="https://github.com/user-attachments/assets/5158c8f9-1e95-4165-9968-5d0abaef1193" />*
*<img width="657" height="253" alt="Aliases of nslookup" src="https://github.com/user-attachments/assets/1f3e546d-9bf3-47ed-9989-f659e292adbf" />*


**What's happening here:** a single DNS query returned **four chained answers** in one response. This is a **CNAME (Canonical Name) chain**- each CNAME record points to another name instead of an IP directly, until the final record resolves to an actual IP address.
This pattern is extremely common with content delivery networks (CDNs) like Akamai ( visible here as 'akamaidege.net'), which use DNS aliasing to route users to the nearest/fastest server.

### Note on Akamai / CDN CNAME Chains

The long CNAME chain observed in the `www.cisco.com` lookup 
(`akadns.net` → `edgekey.net` → `akamaiedge.net`) reflects Cisco's use of 
**Akamai**, a major Content Delivery Network (CDN) provider. Each CNAME hop 
represents a layer of Akamai's traffic-routing system, which dynamically 
resolves the query to the IP address of the edge server geographically closest 
to (or best performing for) the requesting user — rather than a single fixed 
server address. This is why large websites often show multi-hop CNAME chains 
instead of resolving directly to one static IP.


### 3. MX Records - DNS Supporting Email Delivery

DNS doesn't only resolve websites - it also tells mail servers where to deliver email for a domain, via **MX (Mail Exchanger)* records.

*<img width="766" height="524" alt="nslookup email" src="https://github.com/user-attachments/assets/7ea522b9-da63-43dd-aa23-ff2cc815f82c" />*

**Preference values** determine priority - lower numbers are tried first. Multiple MX records provide redundancy : if the primary mail server is unreachable, the sending server falls back to the lowest preference value.

*<img width="953" height="232" alt="Email protection" src="https://github.com/user-attachments/assets/c98d837a-5567-43c6-aa72-1f2598a89d00" />*

This second example shows a domain ('smtafe.wa.edu.au') that outsources its email security to a third party - its MX record points to 'mail.protection.outlook.com', meaning all inbound mail is routed through Microsoft's email filtering service before reaching the organization.

---

### Security prospective

###  DNS as an Attack Surface

- **DNS Spoofing/Cache Poisoning**: an attacker injects a forged DNS response, 
  redirecting a domain to a malicious IP instead of the legitimate one — 
  conceptually similar to ARP spoofing but at the Application Layer
- **Typosquatting**: registering lookalike domains (e.g., `m1crosoftsupport.co`, 
  documented in the phishing triage lab) that DNS resolves without any concept 
  of "this domain looks suspicious" — DNS trusts whatever domain is registered

###  DNS Tunneling

DNS queries can carry arbitrary encoded data in the subdomain portion of a query 
(e.g., `TWFsd2FyZURhdGE.attacker-domain.com`), and DNS traffic (port 53) is 
almost universally allowed through firewalls since normal internet use depends 
on it. Attackers exploit this to exfiltrate stolen data or maintain 
command-and-control (C2) channels while evading traditional network filtering 
that would block other outbound protocols.

**Detection indicators:**
- Abnormally long or high-entropy (random-looking) subdomains
- Unusually high volume of DNS queries to a single external domain in a short 
  time window
- Frequent use of uncommon record types (TXT, NULL) that aren't typical for the 
  domain's apparent purpose
- DNS queries directed to external/unauthorized DNS servers rather than the 
  organization's configured resolver

###  MX Record Reconnaissance

Looking up an organization's MX records is a legitimate administrative task, but 
also a common **reconnaissance step** for attackers planning targeted phishing. 
Knowing which email platform an organization uses (e.g., Microsoft 365, as seen 
in the `smtafe.wa.edu.au` lookup resolving to `mail.protection.outlook.com`) 
allows an attacker to craft a much more convincing phishing email — mimicking 
that specific platform's login page and branding, exactly as seen in the 
typosquatted Microsoft phishing case (Case 04) documented in the SIEM triage lab.

###  Cross-Referencing with the SIEM Triage Lab

This connects directly to real-world analysis already performed in this 
portfolio: the [Phishing Triage using SIEM](../phishing-triage-siem) exercises 
included a case where a typosquatted domain (`m1crosoftsupport.co`) impersonated 
Microsoft's login page — a technique that becomes far more effective once an 
attacker has confirmed via MX record lookup that a target organization actually 
uses Microsoft 365 for email.

### Detection Summary Table

| Threat | Key Indicator | Detection Method |
|---|---|---|
| DNS Spoofing | Sudden IP change for a known domain | Compare resolved IP against threat intel / historical baseline |
| Typosquatting | Lookalike domain names | Domain similarity monitoring, brand protection tools |
| DNS Tunneling | Long/random subdomains, high query volume | DNS traffic analysis, entropy scoring in SIEM |
| MX Recon | Repeated MX queries from external sources (harder to detect, mostly passive) | Threat intel on known scanning infrastructure 
