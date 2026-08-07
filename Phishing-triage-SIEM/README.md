# Phishing  Triage using SIEM

this repository documents hands-on phishing alert triage exercises using a SIEM platform (Splunk), simulating real SOC analyst workflows.

##Methodology
For each alert:
1. **Identify** the alert and its data source (email, firewall, etc.)
2. **Investigate** by cross-referencing raw logs in the SIEM
3. **Correlate** related alerts across multiple data sources when applicable 
4. **Classify** as True Positive / False Positive with documented reasoning
5. **Recommend** remediation actions and extract IOCs using proper defanging notation


## Skills Demonstrated
-Phishing indicator analysis (spoofed domains, typosquatting, URL shorteners)
-Multi-source log correlation (email + firewall)
-IOC extraction and defanging
-SOC-standard incident documentation
