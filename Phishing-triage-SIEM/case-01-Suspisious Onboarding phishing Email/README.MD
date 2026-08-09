## Case 01 : Suspicious Onboarding phishing Email

**Alert:** Inbound email containing suspisious External link (Severity: Medium)

### Summary
An inbound email impersonating an HR onboarding notificatoin was flagged for containing a suspicious external link and an unusual sender domain.


### Evidence

*<img width="867" height="697" alt="1st triage report" src="https://github.com/user-attachments/assets/4c945d60-6469-4bdc-9dfc-22f904099e85" />*
*<img width="1333" height="278" alt="1st_splunk shot" src="https://github.com/user-attachments/assets/00a86a01-e6a0-4a88-9df7-d9a80f8374c3" />*


### Analtsis
- **Sender:** 'onboarding@hrconnex.thm' - a domain unrelated to the recipient's actual orgainzation ('thetrydaily.thm'), inconsistent with a legitmate internal HR communication
- **Embedded link:** '`hxxps://hrconnex[.]thm/onboarding/15400654060/j.garcia` - hosted on the same suspicious domain, using a long numeric path typicial of phishing link obfuscation


### Classification: True positive

### Why Escalation was Required

The URL needed to be detonated in a sandbox environment to confirm intent, and proxy/firewall logs needed to be checked for any outbound connection attempts from the recipient.

### Remeditaion Actions
- Block the sender domain at the email gateway and web proxy/firewall
- Analyze the URL in a sandbox
- Notify the affected user and confirm wheter the link was clicked or credentials were entered

### Indicators of Compromise (IOCs)
- Sender: `onboarding[at]hrconnex[.]thm`
- Malicious URL (defanged): `hxxps://hrconnex[.]thm/onboarding/15400654060/j.garcia`
