## Case 03: Correlated phishing click via Firewall log

**Alert:** Access to Blacklisted External URL Blocked by Firewall (Severity : High)

### Summary
Following the amazon phishing email alert in Case 02, this alert confirms that the targeted user's device (10.20.2.17) attempted to access the same malicious URL - providing direct evidence of user interaction with the phishing link.


### Evidence
*<img width="1532" height="598" alt="3rd triage" src="https://github.com/user-attachments/assets/29a55a52-afbe-4b51-b4a3-5a2bf86eedbb" />*
*<img width="634" height="314" alt="3rd Splunk" src="https://github.com/user-attachments/assets/abadc42d-62d7-43f8-9c8b-1f6ac2f8c698" />*

### Analysis
- Source IP '10.20.2.17' attempted an outbound HTTP connection to 'hxxp://bit[.]ly/3sHkX3da12340'
- This URL matches the indicator identified in the earlier phishing email sent to h.harris@thetrydaily.thm
- The firewall blocked connection, but the attempt itself confirms the user clicked the link

### Classification : True positive

### Why Escalation was Required
Although the connection was blocked, this confirms actual user interaction with a known phishing link - the device user need to be identified and the user need to follow-up

### Remediation Actions
- Confirm the device ownership of 10.20.2.17 and identify the user
- Reinforce security awareness training given confirmed click-through
- Continue monitoring further activity from this host

### Indicators of Compromise (IOCs)
- Source IP: 10.20.2.17
- Destination port:80(HTTP)
- URL (defanged): 'hxxp://bit[.]ly/3sJkX3da12340
- Correlated with: Amazon phishing email (sender: 'urgents[at]amazon[.]biz


<img width="886" height="678" alt="3rd report" src="https://github.com/user-attachments/assets/f77ecac1-784a-4d49-a952-5d2c9898edd3" />
