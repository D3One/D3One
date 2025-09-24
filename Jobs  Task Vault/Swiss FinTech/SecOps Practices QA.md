# Senior SecOps (ELK, SIEM, SOC, EDR, AV, Email, Cisco, M365/Teams, Wi-Fi) — Technical Interview Question Bank 

**How to use this set:** 40 items total — **20 theory (50%)** and **20 practical (50%)**. Each item includes brief **“What good answers include”** guidance. Practical tasks test commands/flags, queries, config review, and quick fixes.

---

## A) Theory — Strategy, SIEM/SOC, Detection, Endpoint, Email, Network, M365/Teams, Wi-Fi (20)

1. **Role of a Senior SecOps in a modern SOC**
   **What good answers include:** Ownership of detection/response, runbooks, SIEM health, telemetry quality, threat-informed defense, metrics (MTTD/MTTR), leading incident commanders, purple teaming with AppSec/IT, and continuous improvement.

2. **Log telemetry quality vs quantity**
   **What good answers include:** Coverage, fidelity, normalization, time sync, deduplication; schema standards (ECS/CEF), parsing/grok, enrichment (GeoIP, asset/identity), and cost controls (tiers/retention).

3. **Use cases vs correlation rules vs analytics models**
   **What good answers include:** Clear hypotheses tied to ATT\&CK techniques; rule logic, thresholds, suppression, tuning, ML/outlier detection where appropriate.

4. **Detection engineering lifecycle**
   **What good answers include:** Ideate → develop → test (simulations) → deploy → monitor drift → maintain → retire; coverage mapping to ATT\&CK; false positive management.

5. **ELK vs “traditional” SIEM pros/cons**
   **What good answers include:** ELK flexibility/cost, need to build parsers/content; SIEM turnkey content/compliance; scaling, retention, hot/warm tiers, alerting, SOAR integrations.

6. **EDR vs AV vs XDR**
   **What good answers include:** AV signature/ML prevention; EDR telemetry + response (isolation, kill, quarantine); XDR cross-domain correlation (identity, email, cloud). Mention response latency and coverage.

7. **Email security baseline**
   **What good answers include:** SPF, DKIM, DMARC (reject/quarantine), anti-phish policies (user impersonation, BEC), Safe Links/Safe Attachments equivalents, VIP protection, mailbox intelligence, external tagging.

8. **BEC vs phishing vs spear-phishing**
   **What good answers include:** BEC often financial wire fraud; spoofed domains, lookalike displays; mitigations: DMARC reject, supplier validation, payment change processes, and mail flow rules.

9. **Cisco campus security pillars**
   **What good answers include:** Segmentation (VLAN/VRF), 802.1X (EAP-TLS), dACLs, DHCP Snooping, Dynamic ARP Inspection, Port Security, uRPF, NetFlow, syslog to SIEM, IPS/Firewall (ASA/FTD), AnyConnect posture/MFA.

10. **Network-based threat containment strategies**
    **What good answers include:** NAC quarantine VLANs, ACL shunts, micro-segmentation, sinkholing, EDR isolate host, mail quarantine, conditional access, SOAR playbooks automating these.

11. **Microsoft 365 Defender suite overview**
    **What good answers include:** MDE (endpoint), MDO (email), MDI (identity), MDA/Defender for Cloud Apps (SaaS), Sentinel (SIEM+SOAR). Advanced hunting with KQL across domains.

12. **IR lifecycle (NIST-style) applied to O365 incident**
    **What good answers include:** Prep (audit, MDO/MDE policies), Detect (alerts, hunting), Contain (revoke tokens, reset creds, isolate), Eradicate (malicious rules, OAuth apps), Recover (user notify), PIR.

13. **Threat intel usage in SOC**
    **What good answers include:** Strategic/operational/tactical; indicator lifecycles, scoring, context (actor/TTPs), automation into SIEM/EDR, and measuring TI value (true positive lift).

14. **Wi-Fi enterprise security**
    **What good answers include:** WPA3-Enterprise, 802.1X/EAP-TLS, certificate-based auth, PMF (802.11w) required, disable WPS, separate guest/IoT SSIDs with firewalled VLANs, rogue AP/WIDS monitoring.

15. **Sysmon and Windows logging essentials**
    **What good answers include:** Windows audit policy + Sysmon for process, network, registry, driver loads; forward via Winlogbeat/agent; tune noise; time sync importance.

16. **Linux telemetry essentials**
    **What good answers include:** auditd/augenrules, journald, eBPF/EDR sensors, bash history protections, sudo logs, file integrity (AIDE), authentication logs.

17. **Prioritizing alerts during ransomware**
    **What good answers include:** Identity/privilege alerts (DCs, KRBTGT), mass encryption patterns, backup tampering, EDR isolation at scale, disable SMBv1, block C2, immutable backups.

18. **KPIs/KRIs that matter in SecOps**
    **What good answers include:** Signal-to-noise, MTTD/MTTR, % alerts with runbook, % auto-closed by SOAR, detection coverage by ATT\&CK, patch SLAs for internet-facing assets.

19. **SOAR benefits and risks**
    **What good answers include:** Speed/consistency; risks: over-automation, scope creep, dangerous actions without guardrails; mitigations: approvals, RBAC, simulation mode.

20. **Data retention & legal**
    **What good answers include:** Retention by data type (PCI/PII), privacy constraints, chain-of-custody for evidence, WORM/immutable logs, export for forensics.

---

## B) Practical — Hands-On Tasks (20)

> Provide a terminal, SIEM, or admin console where possible. Each item lists **expected commands/snippets/queries** and **what good answers include**.

1. **Kibana KQL: failed logons burst**
   **Prompt:** Find users with ≥10 failed logons in 5 minutes.
   **Expected:**
   `event.action:"logon_failed" | stats count() by user, bin(@timestamp, 5m) | where count >= 10` *(KQL variant or Lens/TSVB)*
   **Good answers:** Time-window aggregation and outlier logic; consider unique source IPs.

2. **Elasticsearch DSL filter for specific hosts**
   **Prompt:** Build DSL to fetch firewall denies from hostnames starting with `edge-`.
   **Expected (JSON):** `{"query":{"bool":{"must":[{"match":{"action":"deny"}},{"prefix":{"host.name":"edge-"}}]}}}`
   **Good answers:** Index selection, time range, and output size/scroll.

3. **Logstash grok fix**
   **Prompt:** Parse ASA log: `Deny tcp src 10.1.2.3/54321 dst 10.2.3.4/443`.
   **Expected grok:**
   `%{WORD:action} %{WORD:proto} src %{IP:src_ip}/%{INT:src_port} dst %{IP:dst_ip}/%{INT:dst_port}`
   **Good answers:** Add `ecs` fields via mutate (network.transport, source.*, destination.*).

4. **Sigma rule skeleton → SIEM query**
   **Prompt:** Convert Sigma for suspicious PowerShell (`EncodedCommand`) into your SIEM query.
   **Expected:** KQL/Splunk/ES equivalent (e.g., KQL: `DeviceProcessEvents | where ProcessCommandLine has "EncodedCommand"`).
   **Good answers:** Add exclusions for legitimate tooling; map to ATT\&CK T1059.001.

5. **EDR host isolation**
   **Prompt:** Isolate a Windows host via Defender for Endpoint.
   **Expected:** MDE portal isolate action or PowerShell: `Start-MpWDOScan` (scan), or API call; CrowdStrike example: `cs-falcon hosts contain --ids <ID>`.
   **Good answers:** Note impact (breaks SMB/RDP except EDR), unisolate process.

6. **Windows event triage**
   **Prompt:** Pull last 100 admin logons from Security log.
   **Expected:**
   PowerShell: `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624; StartTime=(Get-Date).AddDays(-1)} | Where-Object {$_.Properties[8].Value -eq '10'} | Select -First 100` *(or filter for LogonType 10 = RDP)*
   **Good answers:** Explain logon types, SID parsing.

7. **Sysmon config tweak**
   **Prompt:** Add network connection include for `powershell.exe` to any external IP.
   **Expected (XML):** `<NetworkConnect onmatch="include"><Image condition="is">C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe</Image><DestinationIsIpv6 condition="is">false</DestinationIsIpv6></NetworkConnect>`
   **Good answers:** Hash/command line filters to reduce noise.

8. **Linux quick triage**
   **Prompt:** List listening processes with PIDs and signed binaries where applicable.
   **Expected:** `ss -lntp`; `lsof -i -P -n`; check packages/signatures (e.g., `rpm -qf $(which sshd)` or `debsig-verify`).
   **Good answers:** Correlate with EDR sensor.

9. **Email auth records**
   **Prompt:** Propose SPF/DKIM/DMARC for `example.com` (hosted at a generic DNS).
   **Expected:**
   SPF: `v=spf1 include:spf.protection.outlook.com -all`
   DMARC: `_dmarc TXT "v=DMARC1; p=reject; rua=mailto:dmarc@example.com; fo=1"`
   **Good answers:** DKIM selector publish, alignment, monitor mode before reject.

10. **Exchange Online transport rule**
    **Prompt:** Tag external emails and block lookalike domains.
    **Expected:** EAC rule: prepend “\[EXTERNAL]”; create condition `Sender domain matches patterns` for lookalikes; or Defender anti-phishing policy with `EnableSimilarDomains`.
    **Good answers:** Avoid tagging internal; exceptions for approved partners.

11. **M365 Defender Advanced Hunting (KQL)**
    **Prompt:** Find suspicious `rundll32` spawning with URL in cmdline.
    **Expected:**

```kql
DeviceProcessEvents
| where FileName =~ "rundll32.exe"
| where ProcessCommandLine has "http"
| project Timestamp, DeviceName, InitiatingProcessFileName, ProcessCommandLine
```

**Good answers:** Join to NetworkEvents for egress; add device group filter.

12. **Sentinel analytic rule threshold**
    **Prompt:** Alert when a single user generates >50 failed logons in 10 minutes across 3+ IPs.
    **Expected (KQL sketch):**

```kql
SigninLogs
| where ResultType != 0
| summarize Count=dcount(IPAddress), Attempts=count() by UserPrincipalName, bin(TimeGenerated, 10m)
| where Attempts > 50 and Count >= 3
```

**Good answers:** Entity mapping, alert suppression window.

13. **Cisco switch hardening checklist**
    **Prompt:** Name commands/features to stop rogue DHCP and ARP spoofing.
    **Expected:** `ip dhcp snooping`, trust uplinks; `ip arp inspection vlan X`, `ip verify source`, storm-control, `switchport port-security`.
    **Good answers:** Log to SIEM, errdisable recovery with care.

14. **ASA/FTD logging to SIEM**
    **Prompt:** Enable syslog and send to 10.0.5.10, level warnings.
    **Expected:**
    ASA:

```
logging enable
logging trap warnings
logging host inside 10.0.5.10
```

**Good answers:** UTC time, `logging device-id`, `logging timestamp`, test with `test syslog`.

15. **Wi-Fi Enterprise configuration**
    **Prompt:** Outline secure setup for corp SSID.
    **Expected:** WPA3-Enterprise, 802.1X EAP-TLS, RADIUS/NPS/ISE, PMF required, separate guest VLAN, disable WPS, per-user certs, dynamic VLANs via dACL.
    **Good answers:** MDM certificate lifecycle, rogue AP detection.

16. **ELK alert for DNS tunneling**
    **Prompt:** Detect excessive TXT queries per host.
    **Expected KQL/ES:** aggregate by `client.ip` and `dns.question.type: "TXT"` with threshold; whitelist known services.
    **Good answers:** Length/entropy checks, domain length heuristic.

17. **SOAR playbook outline**
    **Prompt:** Auto-contain malware alert from EDR.
    **Expected:** Validate alert, gather context, hash reputation, isolate host via EDR API, block hash in AV/EDR, disable account if lateral movement suspected, open ticket, notify owner, require human approval for destructive steps.
    **Good answers:** Rollback isolation on false positive, SLAs.

18. **Windows Defender quick commands**
    **Prompt:** Update signatures, start quick scan, list threats.
    **Expected:**
    `Update-MpSignature`
    `Start-MpScan -ScanType QuickScan`
    `Get-MpThreatDetection`
    **Good answers:** Real-time monitoring state: `Get-MpComputerStatus`.

19. **YARA on a memory dump (concept)**
    **Prompt:** How to scan memdump with YARA and handle hits.
    **Expected:** `yara -r rules.yar memdump.bin`; triage with strings/volatility; quarantine artifact; add IOC to SIEM.
    **Good answers:** Avoid PII leakage, chain-of-custody.

20. **Contain OAuth-app abuse in M365**
    **Prompt:** A malicious consented OAuth app exfiltrates mail. Steps?
    **Expected:** Revoke app’s consent and service principal, investigate audit logs, refresh tokens revoke, user password reset + MFA, DLP checks, create conditional access: block risky OAuth scopes, alert rules.
    **Good answers:** Educate users, publisher verification policies.

---

## Scoring Guidance (quick rubric)

* **Strategy & SOC fundamentals (Theory 1–5):** clarity, practicality (0–10)
* **Detection/telemetry/endpoint/email/network (Theory 6–15):** depth & correctness (0–25)
* **IR/metrics/legal (Theory 16–20):** actionable understanding (0–15)
* **Hands-on fluency (Practical 1–20):** correct queries/flags/configs, safe defaults (0–40)
* **Communication:** concise, priority-driven reasoning (0–10)

---
