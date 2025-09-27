
# 🔐 The 10 Most Common Ways Microsoft-Centric Enterprises Get Popped (and How to Shut the Door)

*You’re already living in AppSec/DevSecOps/Product Security land. This piece gives you the **practical**, threat-informed paths attackers actually use against Windows/AD/Exchange/SQL/SharePoint estates—plus the controls that measurably move risk down without turning you into a full-time red-teamer.*

---

## 1) 📨 Consent Phishing & Token Games in Entra ID (Azure AD)

*Short version:* Attackers skip passwords and **abuse OAuth**: they trick users or admins to grant a malicious app “read mail / read files / send as user,” then live off refresh tokens and weak Conditional Access. Token theft/forgery incidents (think **Storm-0558**) show how identity is the new perimeter.

**How it works in the wild**

* Drive-by consent links, malicious enterprise apps, over-broad permissions.
* Token replay and refresh-token persistence; pivot across M365 workloads (OWA/SharePoint/Teams).
* Real-world case: Storm-0558 forged tokens to access Exchange Online mailboxes. *Lesson:* assume tokens are crown jewels and log accordingly. ([Microsoft][1])

*What to do (right now)*

* **Block user consent** by default; restrict to verified publishers + admin-consented apps only.
* **Conditional Access**: require phishing-resistant MFA for *all* privileged roles; session controls for sensitive scopes.
* **Audit tokens & apps**: hunt risky sign-ins and suspicious app grants; enable enhanced logging. *MFA fatigue (“push bombing”) is still a thing—enable number-matching and rate-limit prompts.* ([CISA][2])

---

## 2) 🔓 Password Spraying & RDP/VPN Misuse

*Short version:* Low-and-slow **password spray** → single foothold → mailbox or VPN portal → internal pivot. Exposed RDP is still ransomware’s favorite front door.

**How it works in the wild**

* Sprays against OWA/Entra ID/RDP; legacy auth & weak lockouts help.
* Many ransomware crews still buy sprayed creds from initial access brokers. ([CISA][3])

*What to do (right now)*

* **Disable legacy/basic auth**, enforce modern auth + MFA (phishing-resistant where possible).
* **Smart lockouts** + password protection (ban common passwords).
* **Shut (or gate) RDP**: disable internet-facing RDP; if unavoidable, put behind ZTNA/PAW jump hosts; alert on anomalous RDP. ([Microsoft Learn][4])

---

## 3) 🪤 LLMNR/NBT-NS Poisoning & NTLM Relay (esp. to AD CS)

*Short version:* Old name-resolution protocols + NTLM = **credential interception** and relays to services like **AD Certificate Services** for instant, hard-to-revoke auth.

**How it works in the wild**

* Responder/relay tools capture or relay NTLM to web enroll endpoints; **PetitPotam** coerces auth, relays to AD CS, issues DC-equivalent certs. Game over. ([Microsoft Support][5])

*What to do (right now)*

* **Kill LLMNR/NBT-NS** via GPO/Intune; **require SMB signing**; enable **Extended Protection for Authentication** on AD CS web enrollment.
* Treat AD CS like Tier-0. Audit templates & enrollment rights; disable web enroll where possible. ([CSO Online][6])

---

## 4) 🎫 Kerberoasting & AS-REP Roasting

*Short version:* AD service accounts with SPNs and weak passwords = **offline hash cracking party**. Turning off pre-auth makes AS-REP roasting trivial.

**How it works in the wild**

* Request TGS tickets (kerberoast) or AS-REP for pre-auth-disabled users; crack offline → escalate.
* Works because service accounts often have static, long-lived creds and broad rights. ([MITRE ATT&CK][7])

*What to do (right now)*

* **Rotate & lengthen** service account passwords (machine-managed if possible).
* **No pre-auth disabled** for human accounts; monitor abnormal TGS request volume.

---

## 5) 🧪 LSASS Dumping → Pass-the-Hash / Pass-the-Ticket

*Short version:* Memory scraping (LSASS) yields creds and Kerberos material; attackers **reuse hashes/tickets** across the domain—no plaintext needed.

**How it works in the wild**

* Credential dumping via userland EDR-evasion, mini-dumps, or DMP files; PTH/PTT to laterally move and escalate.
* Classic, still lethal against flat networks and always-admin users. ([MITRE ATT&CK][8])

*What to do (right now)*

* **LSASS protection** (RunAsPPL), **WDAC/AppLocker**, block unsigned drivers.
* **Tiering/PAW**: no browsing/mail on admin workstations; **local admin randomization with LAPS**.
* Follow Microsoft’s **Mitigating Pass-the-Hash** guidance as a program, not a checkbox. ([Microsoft][9])

---

## 6) 🏛️ AD CS Template Abuse & “Shadow Credentials”

*Short version:* Misconfigured **certificate templates** let attackers enroll certs that map to privileged SIDs (ESC1-ESC8). Certs act like **keys to the kingdom**—and outlive password resets.

**How it works in the wild**

* Any-auth enroll, “ENROLLEE SUPPLIES SUBJECT,” ClientAuth EKU + weak ACLs → request a cert for someone else → log on as them.
* Persistence via long-lived certs; stealth DCSync. ([SpecterOps][10])

*What to do (right now)*

* **Inventory templates & ACLs**; remove risky EKUs/issuer combos; restrict enrollment to controlled groups; log 4886/4887/4890.
* Detect with modern playbooks (Certify/Defender for Identity analytics). ([CrowdStrike][11])

---

## 7) 📮 Exchange: ProxyLogon/ProxyShell & Friends

*Short version:* Historic but still evergreen: **Exchange SSRF/RCE chains** (ProxyLogon/ProxyShell) delivered shells, web shells, and mail theft. Unpatched/hard-to-retire on-prem Exchange remains a top target.

**How it works in the wild**

* CVE-2021-26855 SSRF leads to auth bypass; chained with file-write/RCE for full server takeover. Attackers then harvest mail, drop web shells, pivot inward. ([Microsoft][12])

*What to do (right now)*

* **Be on supported CUs/SUs**; verify no web shells; review the **CISA emergency directives** and top exploited lists—these CVEs are still in KEV.
* If you can: **migrate off** internet-facing on-prem Exchange; isolate and keep patched if you must retain. ([CISA][13])

---

## 8) 📄 SharePoint RCE & Poor Patch Hygiene

*Short version:* Internet-exposed **SharePoint** with old **RCEs** (e.g., CVE-2019-0604) continues to be exploited for initial access and web-shell beachheads; recent SharePoint zero-days remind us this isn’t ancient history.

**How it works in the wild**

* Malicious package upload bypasses checks → code execution → web shell → internal discovery + data theft.
* CISA/NVD have flagged SharePoint RCEs as routinely exploited; patching lag keeps this alive. ([NVD][14])

*What to do (right now)*

* **Take SharePoint off the internet** or put behind WAF/VPN; **patch fast**; monitor for unusual w3wp activity and shell artifacts.

---

## 9) 🖨️ “One-Shot” Windows/DC Vulns (ZeroLogon, PrintNightmare)

*Short version:* Some bugs are so bad they bypass years of hygiene. **ZeroLogon** (Netlogon EoP) and **PrintNightmare** (Print Spooler RCE/LPE) are prime examples.

**How it works in the wild**

* ZeroLogon let attackers reset DC machine account passwords; PrintNightmare enabled RCE/LPE via spooler paths. Both led to rapid domain compromise before patches landed.

*What to do (right now)*

* Treat CISA emergency directives as **Tier-0 break glass**. Inventory DCs/servers for spooler usage; disable where not required; verify the Netlogon hardening state. ([Microsoft][15])

---

## 10) 🗃️ SQL Server Abuse: xp_cmdshell, Linked Servers, Over-Privileged Svc Accounts

*Short version:* Once inside, SQL Server often becomes the **lateral-movement trampoline**: weak service account hygiene + permissive **linked servers** + resurrected **xp_cmdshell** mean shell-on-box and quick domain sprawl.

**How it works in the wild**

* Attackers flip **xp_cmdshell** back on or exploit **RPC OUT** across linked servers to execute OS commands, then harvest local creds and pivot.
* DB service accounts frequently run with **local admin** or domain privileges.

*What to do (right now)*

* Keep **xp_cmdshell** disabled; review `sp_configure` drift.
* **Least privilege** service accounts; rotate and remove local admin rights.
* Audit and prune **linked servers** and cross-DB trusts; monitor for unusual `EXECUTE AT`/external queries. ([Microsoft Support][5])

---

## 🧭 Blue-Team Quick Wins (that pay off fast)

*These are the “do them first” controls that cut across many paths and won’t require a PhD in reverse engineering.*

* **Identity & MFA**

  * Block user consent; **app governance** for OAuth; enforce MFA with **number-matching**; Conditional Access for risky sign-ins. ([CISA][2])
* **Kill legacy auth & exposure**

  * Disable basic/legacy protocols; **no internet-exposed RDP**; Exchange/SharePoint behind WAF/ZTNA if they must be public. ([CISA][16])
* **Credential hygiene**

  * **Windows LAPS** for unique local admin passwords (workstations **and** servers/DCs); protect LSASS; tiered admin model. ([Microsoft Learn][17])
* **Network & Protocol hardening**

  * Disable **LLMNR/NBT-NS**; require SMB signing; EPA on AD CS; aggressively segment Tier-0 assets. ([Microsoft Support][5])
* **Patch the “greatest hits”**

  * ZeroLogon, PrintNightmare, ProxyLogon/ProxyShell, SharePoint RCEs—all still targeted wherever they linger. ([CISA][18])
* **Detection engineering**

  * Alert on: new OAuth app grants; unusual TGS request spikes; Entra risky sign-ins; AD CS enroll anomalies; web-shell patterns on IIS/Exchange/SharePoint.

*Pro tip:* **Measure**. Track “mean time to revoke risky OAuth grants,” “% of LAPS-covered endpoints,” and “% of endpoints with LLMNR/NBT-NS disabled” as your north-star lagging indicators.

---

## 🧱 What to prioritize in the next 30/60/90

* **30 days:** block user consent; enable number-matching; disable LLMNR/NBT-NS; deploy LAPS to Tier-0/Tier-1; verify Exchange/SharePoint patch level and internet exposure. ([Microsoft Learn][17])
* **60 days:** AD CS review (templates/ACLs/EPA); password spray detections; remove internet-facing RDP; service account rotation. ([SpecterOps][10])
* **90 days:** implement admin tiering & PAWs; WDAC/AppLocker on Tier-0; full Conditional Access coverage incl. device risk.

---

## 📚 References & Further Reading

* **Microsoft** — *Storm-0558 token-forgery analysis*; *HAFNIUM/ProxyLogon*; *Mitigating Pass-the-Hash*; *Windows LAPS overview*. ([Microsoft][1])
* **CISA** — *Enhanced monitoring for Exchange Online incident*; *Password spraying playbooks/tools*; *RDP controls*; *Top exploited vulns (ProxyLogon/ProxyShell/ZeroLogon)*; *PetitPotam/KB5005413*. ([CISA][19])
* **MITRE ATT&CK** — *Kerberoasting (T1558.003)*; *AS-REP Roasting (T1558.004)*; *PowerShell (T1059.001)*. ([MITRE ATT&CK][7])
* **SpecterOps** — *Certified Pre-Owned (AD CS abuses)*. ([SpecterOps][10])
* **NVD/CISA KEV** — *SharePoint CVE-2019-0604 RCE*. ([NVD][14])

---

[1]: https://www.microsoft.com/en-us/security/blog/2023/07/14/analysis-of-storm-0558-techniques-for-unauthorized-email-access/?utm_source=chatgpt.com "Analysis of Storm-0558 techniques for unauthorized email ..."
[2]: https://www.cisa.gov/sites/default/files/publications/fact-sheet-implement-number-matching-in-mfa-applications-508c.pdf?utm_source=chatgpt.com "Implementing Number Matching in MFA Applications"
[3]: https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-136a?utm_source=chatgpt.com "StopRansomware: BianLian Ransomware Group"
[4]: https://learn.microsoft.com/en-us/security/operations/incident-response-playbook-password-spray?utm_source=chatgpt.com "Password spray investigation"
[5]: https://support.microsoft.com/en-us/topic/kb5005413-mitigating-ntlm-relay-attacks-on-active-directory-certificate-services-ad-cs-3612b773-4043-4aa9-b23d-b87910cd3429?utm_source=chatgpt.com "KB5005413: Mitigating NTLM Relay Attacks on Active ..."
[6]: https://www.csoonline.com/article/568015/how-to-disable-llmnr-in-windows-server.html?utm_source=chatgpt.com "How to disable LLMNR in Windows Server - CSO Online"
[7]: https://attack.mitre.org/techniques/T1558/003/?utm_source=chatgpt.com "Steal or Forge Kerberos Tickets: Kerberoasting"
[8]: https://attack.mitre.org/techniques/T1003/?utm_source=chatgpt.com "OS Credential Dumping, Technique T1003 - Enterprise"
[9]: https://www.microsoft.com/en-us/download/details.aspx?id=36036&utm_source=chatgpt.com "Mitigating Pass-the-Hash (PtH) Attacks and Other ..."
[10]: https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf?utm_source=chatgpt.com "Certified Pre-Owned"
[11]: https://www.crowdstrike.com/wp-content/uploads/2023/12/investigating-active-directory-certificate-abuse.pdf?utm_source=chatgpt.com "Investigating Active Directory Certificate Services Abuse"
[12]: https://www.microsoft.com/en-us/security/blog/2021/03/02/hafnium-targeting-exchange-servers/?utm_source=chatgpt.com "HAFNIUM targeting Exchange Servers with 0-day exploits"
[13]: https://www.cisa.gov/news-events/cybersecurity-advisories/aa21-062a?utm_source=chatgpt.com "Mitigate Microsoft Exchange Server Vulnerabilities"
[14]: https://nvd.nist.gov/vuln/detail/cve-2019-0604?utm_source=chatgpt.com "CVE-2019-0604 Detail - NVD"
[15]: https://www.microsoft.com/en-us/msrc/blog/2021/07/clarified-guidance-for-cve-2021-34527-windows-print-spooler-vulnerability?utm_source=chatgpt.com "Clarified Guidance for CVE-2021-34527 Windows Print ..."
[16]: https://www.cisa.gov/eviction-strategies-tool/info-countermeasures/CM0025?utm_source=chatgpt.com "| CISA"
[17]: https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview?utm_source=chatgpt.com "Windows LAPS overview"
[18]: https://www.cisa.gov/news-events/cybersecurity-advisories/aa22-117a?utm_source=chatgpt.com "2021 Top Routinely Exploited Vulnerabilities"
[19]: https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-193a?utm_source=chatgpt.com "Enhanced Monitoring to Detect APT Activity Targeting ..."

---
