# 🛡️ SOC Analyst Investigation - Phishing Detection

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-red)
![SIEM](https://img.shields.io/badge/SIEM-Splunk-green)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Type](https://img.shields.io/badge/Type-SOC%20Simulation-blue)

---

## Overview

Completed a hands-on SOC Analyst simulation investigating real-world phishing scenarios using Splunk SIEM. Performed full alert triage, log analysis, threat classification, and incident reporting following industry standard SOC playbooks.

This project demonstrates my ability to think and operate as a Tier 1 SOC Analyst from receiving an alert all the way through to writing a professional case report.

![SOC Dashboard](images/soc-dashboard.png)
---

## Tools Used

| Tool | Purpose |
|------|---------|
| Splunk SIEM | Log analysis and threat hunting |
| SOC Simulator (TryHackMe) | Alert triage simulation environment |
| Firewall Logs | Connection verification |
| Email Logs | Phishing investigation |
| Asset Inventory | Employee and host correlation |

---

## Skills Demonstrated

- Alert triage and prioritisation
- Splunk SIEM log analysis and query building
- Phishing email investigation and analysis
- IOC (Indicator of Compromise) identification
- Firewall log correlation
- Asset correlation (IP to employee mapping)
- Incident classification (True Positive / False Positive)
- Escalation decision making
- Professional SOC case report writing

---

## Investigation Information

| Alert ID | Type | Severity | Classification | Escalation Required |
|----------|------|----------|---------------|-------------------|
| 8814 | Phishing Email - Suspicious External Link | Medium | True Positive | No |

![alert information](images/alert-information.png)

---

### Step 1 - Review The Alert
- Read the alert description and understand what triggered it
- Extract all IOCs (IPs, domains, URLs, email addresses)
- Cross reference affected entities with the asset inventory
- Form an initial Understanding

### Step 2 - Investigate in Splunk
- Build targeted queries based on the hypothesis
- Search email logs to confirm what arrived
- Search firewall logs to confirm what was clicked or connected
- Correlate findings across multiple log sources

### Step 3 - Write Case Report
- Classify as True Positive or False Positive with evidence
- Decide if escalation is required
- Document all IOCs found
- Recommend remediation actions

---

##  Alert 8814 - Phishing Email Investigation

### Alert Details

| Field | Value |
|-------|-------|
| Alert ID | 8814 |
| Alert Rule | Inbound Email Containing Suspicious External Link |
| Severity | Medium |
| Type | Phishing |
| Date | 03/11/2026 12:16:28 |

---

**3. Splunk Email log results for hrconnex.thm**
 
Query used: `index=* "hrconnex.thm"`
 
This broad search returned 3 events all from the email datasource. This confirmed the phishing email arrived in Julia's inbox. No firewall or proxy logs were found for this domain at this stage.
 
![Splunk Email Logs](images/Splunk-1.png)
 
---
 
**4. Splunk Firewall logs confirming no connection from Julia's machine**
 
Two queries were used together to reach a conclusive finding:
 
**Query 1 Did Julia click the link?**
 
`index=* SourceIP="10.20.2.8" "hrconnex.thm" datasource="firewall"`
 
This searched specifically for firewall logs showing Julia's machine (`10.20.2.8`) making any connection to `hrconnex.thm`. Result: 0 events no connection was recorded.
 ![Splunk Firewall Logs](images/no-logs.png)
**Query 2 Is firewall logging actually working?**
 
`index=* datasource="firewall"`
 
Before concluding Julia didn't click, I verified that the firewall was actively recording logs. Result: 42 firewall events found confirming the firewall is fully active and recording all connections across the network.
 
**Combined Conclusion:** Since the firewall is confirmed working and recorded 42 other connections, the absence of any connection from Julia's machine to `hrconnex.thm` is conclusive proof that Julia did not click the link.
 
![Splunk Firewall Logs](images/firewall-logging.png)


### Email Analysis

When I opened the alert, I found the following email details:

| Field | Value |
|-------|-------|
| Sender | onboarding@hrconnex.thm |
| Recipient | j.garcia@thetrydaily.thm |
| Subject | Action Required: Finalize Your Onboarding Profile |
| Attachment | None |
| Suspicious URL | https://hrconnex.thm/onboarding/15400654060/j.garcia |
| Direction | Inbound |

---

### Red Flags Identified

Before touching Splunk, I analysed the email and identified these red flags:

**1. Sender Domain Mismatch**
- Company domain = `thetrydaily.thm`
- Sender domain = `hrconnex.thm`
- A legitimate HR email would come from within the company domain

**2. Spear Phishing Indicator**
- The malicious URL contained the victim's username: `/j.garcia`
- This means the attacker specifically targeted Julia Garcia
- This is called Spear Phishing a targeted attack on one specific person

**3. Urgency Tactic**
- Subject line says "Action Required"
- Email body pressures recipient to act immediately
- Classic social engineering tactic to bypass critical thinking

**4. External Domain in Link**
- The link redirects to `hrconnex.thm` — a completely external domain
- A legitimate onboarding link would point to `thetrydaily.thm`
- The random number `15400654060` in the URL is likely a tracking ID to identify who clicked

---

### Asset Correlation

Cross referenced the recipient with the company asset inventory:

| Field | Value |
|-------|-------|
| Full Name | Julia Garcia |
| Department | Content |
| Email | j.garcia@thetrydaily.thm |
| Hostname | win-3452 |
| IP Address | 10.20.2.8 |

> **Why this matters:** Splunk logs don't say "Julia clicked a link." They say "10.20.2.8 made a connection." Asset correlation lets me translate technical identifiers back to real people and machines.

#### Investigation Conclusion

| Query | Purpose | Result |
|-------|---------|--------|
| `index=* "hrconnex.thm"` | Find all logs for domain | 3 email events found |
| `index=* SourceIP="10.20.2.8" "hrconnex.thm" datasource="firewall"` | Did Julia click? | 0 events |
| `index=* datasource="firewall"` | Is firewall working? | 42 events confirmed |

**Final Conclusion:** Julia did not click the link. Proven by zero firewall connections from her machine, with firewall logging confirmed active.

###  Case Report

---

**Time of Activity:**
03/11/2026 12:16:28

---

**List of Affected Entities:**
Julia Garcia | Content Department | Host: win-3452 | IP: 10.20.2.8

---

**Reason for Classifying as True Positive:**
This alert is classified as True Positive because a phishing email was sent from `onboarding@hrconnex.thm` to `j.garcia@thetrydaily.thm` containing a malicious URL `https://hrconnex.thm/onboarding/15400654060/j.garcia`. The email impersonated thetrydaily.thm company HR and created urgency by saying "please complete your final profile setup so we can configure your access."

---

**Reason for Not Escalating:**
Escalation is not required because Julia did not click the link. This was confirmed by searching firewall logs for connections from `10.20.2.8` to `hrconnex.thm` which returned no results, confirming no connection was established between Julia's machine and the attacker's domain. No unauthorised access occurred.

---

**Recommended Remediation Actions:**
- Block domain `hrconnex.thm` at the email gateway level
- Inform Julia Garcia about the phishing attempt and advise her to remain vigilant
- Alert all other team members about emails coming from this domain
- Alert the security team to monitor for further activity from this domain

---

**List of Attack Indicators (IOCs):**

| Indicator Type | Value |
|---------------|-------|
| Sender Email | onboarding@hrconnex.thm |
| Malicious URL | https://hrconnex.thm/onboarding/15400654060/j.garcia |
| Attacker Domain | hrconnex.thm |
| Tactic Used | Spear Phishing |
| Social Engineering | Urgency  "Action Required" subject line |

---

*This project is part of my ongoing cybersecurity portfolio documenting hands-on security investigations and analysis.*
