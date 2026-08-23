# TryHackMe – SOC Level 1 Path: Lab Notes

## Room 1: Junior SOC Analyst (Web App)

**Scenario:** Simulated SIEM dashboard for a company's alert triage workflow, covering the full cycle from detection to remediation.

### Step 1 — Alert Triage (SIEM Dashboard)
The alert queue showed several entries of varying severity. Two **Critical** alerts stood out, both tied to the same source:
<img width="1342" height="644" alt="soc1" src="https://github.com/user-attachments/assets/1ebad5d7-68eb-4445-a49a-0ff78ef5c9a7" />


| Date | Severity | Message |
|---|---|---|
| Aug 23 2026, 16:13 | Critical | Successful SSH login from the suspicious IP `221.181.185.159` |
| Aug 23 2026, 16:09 | Critical | Unauthorized login attempts from IP `221.181.185.159` to port 22 |

The lower-severity alerts (failed logins, expired service account password) were noise by comparison — a **successful** authentication from a flagged IP is what actually warrants escalation, since a failed attempt alone is lower risk.

**Malicious IP identified:** `221.181.185.159`

### Step 2 — Threat Intelligence Lookup (IP Hunter)
<img width="662" height="611" alt="soc2" src="https://github.com/user-attachments/assets/3b6e92c4-ead0-4ad0-8d5b-b27bccfc8431" />



Ran the IP through the built-in IP Hunter tool (standing in for real-world tools like AbuseIPDB or Cisco Talos Intelligence). Result:

- **Verdict:** Malicious — involved in 4 prior cyber attacks
- **ISP:** China Mobile Communications Corporation
- **Domain:** chinamobile.thm
- **Country:** China
- **Categories:** Port Scan, C2 Server, PlugX

The **C2 Server / PlugX** tags are the key detail — PlugX is a known remote-access trojan (RAT) associated with China-linked APT activity, often delivered via spear-phishing and used for persistent remote access. That context upgrades this from "suspicious login" to "likely compromised host talking to a C2 infrastructure."

### Step 3 — Escalation
<img width="636" height="623" alt="soc3" src="https://github.com/user-attachments/assets/055128a0-23a7-4a30-8df2-1d5a59dc733a" />


With a confirmed successful authentication from a known-malicious, C2-linked IP, the next move was to escalate rather than sit on it. Of the four people offered:

- Will Griffin — **SOC Team Lead** 
- Carolyn Stone — Security Architect
- Nadia Watson — Python Developer
- Gideon Gean — Sales Executive

**Escalated to Will Griffin.** The lesson here: escalation should go to whoever owns triage/response decisions for your team (the SOC Team Lead), not to an adjacent technical role or someone outside security entirely.

### Step 4 — Containment (Firewall Block List)
<img width="631" height="546" alt="soc4" src="https://github.com/user-attachments/assets/183030be-f2ed-4869-b3c3-25ca90d9fee8" />

Final step was blocking the IP on the firewall block list, with a comment describing the reason (successful SSH login from an unauthorized IP). Submitting the block produced the room's completion flag:

```
THM{until-we-meet-again}
```


---

## Room 2: SOC Role in Blue Team (Web App)

**Scenario:** A drag-and-drop exercise matching seven security incidents/requests to the correct blue-team role. Available roles: Lucas, Susan, Nick, Ben, Robert, Eugen, Alice.

| Scenario | Correct Role |
<img width="710" height="538" alt="soc6" src="https://github.com/user-attachments/assets/4cbe0325-2ece-4a9e-8f42-275b7c9f6929" />

|---|---|
| SIEM alert about firewall brute-force — who should triage it? | **Lucas — SOC L1 Analyst** |
| HR manager's machine was hit by phishing malware — who does the deep analysis? | **Susan — SOC L2 Analyst** |
| France office hit by ransomware, immediate response needed | **Robert — CERT Lead** |
| Servers storing credit card data need a PCI DSS audit | **Nick — GRC Auditor** |
| Who checks the new tryhackme.thm release for vulnerabilities? | **Ben — Penetration Tester** |
| SIEM is down due to a storage limit — who investigates? | **Eugen — SOC Engineer** |
| FIN7 threat group is actively targeting the company — who analyzes their tactics? | **Alice — Threat Researcher** |

### Role logic
- **L1 vs L2 Analyst:** L1 handles first-line triage of raw alerts; L2 does the deeper investigation once something's confirmed malicious (e.g., actual malware execution, not just an alert).
- **CERT Lead:** owns active incident response for major, already-confirmed incidents (ransomware) — distinct from day-to-day SOC triage.
- **GRC Auditor:** compliance-driven work (PCI DSS, audits, frameworks) rather than hands-on technical response.
- **Penetration Tester:** proactive vulnerability assessment of new releases/infrastructure, not incident response.
- **SOC Engineer:** keeps the tooling itself running (SIEM infrastructure, storage, ingestion) — an "ops" role behind the analysts.
- **Threat Researcher:** tracks specific threat actors/groups (TTPs, campaigns) like FIN7, feeding intel back to the rest of the team.

