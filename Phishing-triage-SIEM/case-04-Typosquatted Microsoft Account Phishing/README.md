## Case 04: Typosquatted Microsoft Account Phishing

**Alert:** Inbound Email Containing Suspicious External Link (Severity: Medium)

### Summary
An email impersonating a Microsoft account security alert used a typosquatted 
domain and a fabricated "unusual sign-in" scenario to lure the recipient into 
clicking a credential-harvesting link.

### Evidence
*<img width="933" height="700" alt="4th report" src="https://github.com/user-attachments/assets/acf510c5-cf84-4c03-8fd1-6f931cf2b9b5" />*
*<img width="1598" height="277" alt="4th splunk" src="https://github.com/user-attachments/assets/3cc9b32f-d704-4e09-9c0a-7ea03b61b760" />*



### Analysis
- **Sender domain:** `m1crosoftsupport.co` — a typosquat of `microsoft.com`, 
  substituting the letter "i" with the numeral "1" to visually mimic the 
  legitimate brand while evading simple domain-matching filters
- **Embedded link:** `hxxps://m1crosoftsupport[.]co/login` — uses the same 
  spoofed domain, strongly suggesting a fake login page designed to harvest 
  credentials
- **Social engineering hook:** the email fabricates a specific, alarming detail 
  (login attempt from Lagos, Nigeria) to provoke an urgent, unverified click

### Classification: True Positive

### Why Escalation Was Required
The URL needed to be opened in a sandboxed environment to confirm whether it 
hosts a fake login page designed for credential harvesting.

### Remediation Actions
- Block the sender domain
- Analyze the URL in a sandbox to confirm phishing page hosting
- Alert other employees about this typosquatted domain to prevent further clicks

### Indicators of Compromise (IOCs)
- Sender email: `no-reply[at]m1crosoftsupport[.]co`
- Sender domain: `m1crosoftsupport[.]co` (typosquat of `microsoft.com`)
- Malicious URL (defanged): `hxxps://m1crosoftsupport[.]co/login`
