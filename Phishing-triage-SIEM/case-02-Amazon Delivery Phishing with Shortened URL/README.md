## Case 02: Amazon Delivery Phishing with Shortened URL

**Alert:** Inbound Email Containing Suspicious External Link (Severity: Medium)

### Summary
An email impersonating an Amazon delivery notification used urgency ("48 hours") 
and a shortened URL to obscure its actual destination — a common technique to 
evade both human suspicion and automated URL filtering.

### Evidence
*<img width="905" height="704" alt="2nd report" src="https://github.com/user-attachments/assets/c09c4b2f-27dc-48eb-93e8-9260c53f0718" />*
*<img width="1340" height="278" alt="2nd splunk" src="https://github.com/user-attachments/assets/e75c9e92-a7bf-434b-9169-0898a4ba4584" />*

### Analysis

- **Sender domain:** `amazon.biz` — not a legitimate Amazon-owned domain 
  (Amazon does not use `.biz` TLDs), indicating brand impersonation
- **URL shortener (bit.ly):** hides the real destination link, preventing 
  immediate assessment of intent without expanding it first — a common evasion 
  technique against automated URL filtering
- **Urgency tactic:** the "48-hour" deadline is a classic social engineering 
  pressure technique designed to reduce careful scrutiny before clicking

### Classification: True Positive

### Why Escalation Was Required
The shortened URL needed to be expanded and analyzed in a sandboxed environment, 
and proxy/firewall logs needed to be reviewed for any outbound connection 
attempts matching this link.

### Remediation Actions
- Block the sender domain
- Expand and analyze the shortened URL in a sandboxed environment
- Monitor for any outbound traffic matching the resolved destination

### Indicators of Compromise (IOCs)
- Sender email: `urgents[at]amazon[.]biz`
- Sender domain: `amazon[.]biz`
- Malicious URL (defanged, shortened): `hxxp://bit[.]ly/3sHkX3da12340`
