
# SOC Use Cases for an Open-Source Stack on AWS (Financial Exchange | Swiss Fintech | Ivan Piskunov edited)

**Stack assumptions:** Linux (Ubuntu/Amazon Linux), Apache httpd, NGINX, Istio service mesh, containers (EKS/ECS), AWS native logs (CloudTrail, VPC Flow Logs, ALB/ELB/WAF where applicable). Open-source SOC tooling: **ELK (Elastic Security), Wazuh**, **TheHive** (IR/case mgmt), **Cortex** (enrichment), **MISP** (threat intel). *If you meant “MISSP,” we assume **MISP** (Malware Information Sharing Platform).*

---

## Chapter A — Identity, Access & Account Security (AWS)

### Use Case 1 — Console login **without MFA**

**Why:** High-risk sign-in for exchange operators/admins.
**Signals (ELK):** CloudTrail `ConsoleLogin` + `additionalEventData.MFAUsed:"No"`.
**Mini example (KQL):**

```
event.dataset : "aws.cloudtrail" and event.action : "ConsoleLogin" and
aws.cloudtrail.user_identity.type : "IAMUser" and
aws.cloudtrail.response_elements.MFAAuthenticated : "false"
```

**Response:** Auto-open a TheHive case; Cortex pulls MISP IOCs for the source IP; Wazuh pushes host-level context for the user’s last SSH. Force password reset and attach IAM policy diff.
**Standards & best practices:** NIST CSF 2.0 (Protect/Detect/Respond); AWS Well-Architected Security Pillar (MFA, strong identity); CIS Controls (Access Control). ([NIST Publications][1])

---

### Use Case 2 — Privilege escalation via **Inline/Attached Admin policy**

**Why:** Sudden grant of `AdministratorAccess` or wildcard `*:*`.
**Signals:** CloudTrail `PutUserPolicy|AttachUserPolicy|CreatePolicyVersion`.
**Mini example (KQL):**

```
event.dataset:"aws.cloudtrail" and event.action:("AttachUserPolicy" or "PutUserPolicy" or "CreatePolicyVersion")
and aws.cloudtrail.request_parameters.policyDocument.permissionsBoundary:* 
```

**Response:** TheHive playbook: snapshot current permissions, detach policy, notify owner, require break-glass approval.
**Standards:** NIST SP 800-53 (AC-2, AC-6), AWS Security Hub “Foundational Best Practices.” ([NIST Publications][2])

---

### Use Case 3 — **Root user** access key or console activity

**Why:** Root keys/console should be unused.
**Signals:** CloudTrail `CreateAccessKey` with `userIdentity.type:"Root"`, any `ConsoleLogin` by Root.
**Mini example (KQL):**

```
aws.cloudtrail.user_identity.type:"Root" and (event.action:"CreateAccessKey" or event.action:"ConsoleLogin")
```

**Response:** Immediate TheHive P1; auto-page on-call; IAM block key, enable MFA on root; post-incident hardening checklist.
**Standards:** NIST CSF 2.0 (Govern/Protect), AWS Well-Architected (root account protections). ([NIST][3])

---

### Use Case 4 — **CloudTrail tampering** (Stop logging / trail deletion)

**Why:** Classic anti-forensics.
**Signals:** CloudTrail `StopLogging|DeleteTrail|UpdateTrail` targeting org trails.
**Mini example (Elastic prebuilt idea):** Alert on `aws.cloudtrail.eventName:StopLogging` for org trail; pivot to S3 trail bucket access logs.
**Response:** Auto-enable org trail, block offending principal, forensics timeline in TheHive.
**Standards:** NIST 800-53 (AU-2/AU-12/AU-6), Elastic detection rules guidance. ([NIST Publications][2])

---

### Use Case 5 — **GuardDuty high-severity finding** ingestion & triage

**Why:** Managed threat intel on AWS abuse, crypto, exfil.
**Signals:** Wazuh → GuardDuty integration; Elastic parses `severity>=7`.
**Mini example:** Wazuh AWS module ingests GuardDuty S3/SNS; TheHive case auto-created with mapped MITRE tactics.
**Response:** Enrichment (Cortex/MISP), quarantine instance, rotate creds, stamp out persistence.
**Standards:** AWS GuardDuty + Wazuh integration docs; NIST CSF Respond. ([Wazuh Documentation][4])

---

### Use Case 6 — **Anomalous geo/IP** for API keys or console

**Why:** Sudden sign-ins from atypical ASN/country.
**Signals:** CloudTrail + GeoIP in Elastic; velocity & impossible travel logic.
**Mini example (KQL skeleton):**

```
event.dataset:"aws.cloudtrail" and event.action:"ConsoleLogin"
| by user.name
| detect geo_distance_km(previous(ip), ip) > 3000 within 1h
```

**Response:** Step-up verification; short-lived deny policy; IR per NIST 800-61.
**Standards:** NIST 800-61 (IR process), NIST CSF Detect. ([NIST Publications][5])

---

## Chapter B — Network, Perimeter & Traffic (VPC/ALB/NLB/Edge)

### Use Case 7 — **VPC Flow Logs beaconing** to rare external /24

**Why:** C2/exfil against exchange back-office nodes.
**Signals:** VPC Flow Logs (S3/CloudWatch) → Filebeat AWS module → Elastic; look for periodic low-byte egress to rare ASN.
**Mini example:** Elastic job: count distinct 5-min intervals with small `bytes_out` to same dst /24 over 24h.
**Response:** Tag IOC in MISP; Istio/EgressPolicy block; case to TheHive.
**Standards:** AWS VPC Flow Logs + Elastic AWS module docs; NIST CSF Detect. ([AWS Documentation][6])

---

### Use Case 8 — **Security Group opened to 0.0.0.0/0** on sensitive ports

**Why:** Port exposure (22/3306/6379) on trading systems.
**Signals:** CloudTrail `AuthorizeSecurityGroupIngress` with `cidrIp:"0.0.0.0/0"`.
**Mini example (KQL):**

```
event.dataset:"aws.cloudtrail" and event.action:"AuthorizeSecurityGroupIngress" and
aws.cloudtrail.request_parameters.ipRanges.items{}.cidrIp:"0.0.0.0/0" and
aws.cloudtrail.request_parameters.fromPort: (22 or 3306 or 6379)
```

**Response:** Auto-revert rule; open P2 in TheHive; run impact scan.
**Standards:** AWS Security Pillar (network segmentation), CIS Controls v8 (Secure Config). ([AWS Documentation][7])

---

### Use Case 9 — **ALB/NGINX 5xx surge** (DoS or app fault)

**Why:** Exchange front ends must be highly reliable.
**Signals:** ALB access logs + NGINX error logs into Elastic; anomaly detection on 5xx rate.
**Mini example (KQL):**

```
(event.dataset:"nginx.error" and http.response.status_code:[500 TO 599]) or
(event.dataset:"aws.elb" and http.response.status_code:[500 TO 599])
```

**Response:** Scale out, block abusive IP ranges, root-cause in app team; TheHive problem ticket.
**Standards:** NGINX security controls (TLS/upstream hardening); Apache tips if using httpd. ([NGINX Documentation][8])

---

### Use Case 10 — **Malicious payload patterns** in proxy logs

**Why:** SQLi/LFI/proto smuggling against order APIs.
**Signals:** NGINX/Apache access logs; regex on `'union select'`, `../../`, abnormal `Transfer-Encoding`.
**Mini example (Wazuh rule):**

```xml
<rule id="850001" level="10">
  <if_group>nginx</if_group>
  <match>(\.\./\.\.|union\s+select|bash -i >& /dev)</match>
  <description>Web attack pattern in proxy log</description>
</rule>
```

**Response:** Block IP via NGINX map; push indicators to MISP; inform app team.
**Standards:** NIST 800-53 (SI-4), CIS Controls v8 (Protective Controls). ([NIST Publications][2])

---

### Use Case 11 — **Flow Logs deletion**

**Why:** Blind SOC = blind attacker’s dream.
**Signals:** CloudTrail `DeleteFlowLogs`.
**Mini example:** Use Elastic prebuilt rule “AWS VPC Flow Logs Deletion” as template.
**Response:** Recreate Flow Log, block actor, forensics.
**Standards:** Elastic detection rules explorer; AWS VPC docs. ([detection.fyi][9])

---

## Chapter C — Compute, OS & Endpoint (Linux)

### Use Case 12 — **Unauthorized sudoers change**

**Why:** Stealth escalation on trade hosts.
**Signals:** Wazuh FIM watches `/etc/sudoers`, `/etc/sudoers.d/*`.
**Mini example (Wazuh syscheck):**

```xml
<syscheck>
  <directories check_all="yes">/etc/sudoers,/etc/sudoers.d</directories>
</syscheck>
```

**Response:** Revert from golden image; rotate local creds; timeline in TheHive.
**Standards:** Wazuh FIM; NIST 800-53 (CM-5/CM-6). ([Wazuh Documentation][10])

---

### Use Case 13 — **Suspicious SSH source (new ASN/country)**

**Why:** Stolen keys to bastions.
**Signals:** Linux auth logs via Wazuh + GeoIP enrichment in Elastic; velocity & ASN change.
**Mini example (KQL):**

```
event.dataset:"system.auth" and system.auth.ssh.event:"Accepted" 
| by host.name, user.name
| alert when new ASN for last 30d
```

**Response:** Disable key, require MFA/SSO, collect TTY history, isolate host.
**Standards:** NIST CSF Detect/Respond. ([NIST Publications][1])

---

### Use Case 14 — **Crypto-miner process** detection

**Why:** Resource theft/perf degradation; possible compromise.
**Signals:** Wazuh Rootcheck + process list for `xmrig`, high CPU long-lived proc, suspicious outbound to mining pools.
**Mini example:** Wazuh rootcheck every 12h; Elastic ML job on CPU > 90% with network egress to known pools.
**Response:** Kill proc, reimage, investigate initial access, add IOCs to MISP.
**Standards:** Wazuh Rootcheck docs. ([Wazuh Documentation][11])

---

### Use Case 15 — **FIM on web stack & app code**

**Why:** Backdoor via `nginx.conf`, `httpd.conf`, `/var/www`.
**Signals:** Wazuh FIM baseline & real-time alerts; Elastic correlates with deploy window.
**Mini example:** Add `/etc/nginx`, `/etc/httpd`, `/var/www/*` to syscheck; hash & owner changes alert.
**Response:** Rollback with IaC; diff changed files; create TheHive case.
**Standards:** Wazuh FIM PoC/usage; ISO 27001 Annex A 8.16 (monitoring). ([Wazuh Documentation][12])

---

### Use Case 16 — **Rootkit behavior** (hidden files/ports/procs)

**Why:** Stealthy persistence.
**Signals:** Wazuh Rootcheck anomalies; optional test via Reptile rootkit lab.
**Mini example:** Enable `<rootcheck>` in `ossec.conf`; alert on hidden process/ports.
**Response:** Isolate, capture memory, reimage, credential reset, threat hunt lateral movement.
**Standards:** Wazuh rootcheck guides/blog. ([Wazuh Documentation][13])

---

### Use Case 17 — **CIS hardening drift** (Linux/Docker)

**Why:** Baseline compliance & attack surface.
**Signals:** Wazuh SCA against CIS Benchmarks (Linux, Docker).
**Mini example:** Enable SCA; alert when critical checks fail (e.g., SSH root login permitted).
**Response:** Ticket misconfig findings; verify with change advisory; attest in Audit Manager.
**Standards:** Wazuh SCA & CIS references. ([Wazuh Documentation][14])

---

## Chapter D — Application, Proxy & Service Mesh (Istio)

### Use Case 18 — **Istio mTLS not enforced (Permissive mode observed)**

**Why:** Plaintext east-west traffic in a mesh.
**Signals:** Istio telemetry shows plaintext; config is `PERMISSIVE` instead of `STRICT`.
**Mini example (YAML hardening):**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
spec:
  mtls:
    mode: STRICT
```

**Response:** Enforce STRICT mTLS, roll through namespaces; alert on plaintext attempts.
**Standards:** Istio security best practices (mTLS strict). ([Istio][15])

---

### Use Case 19 — **JWT anomalies in Istio** (bad issuer/claims)

**Why:** API auth bypass attempts.
**Signals:** Istio `RequestAuthentication` failures; Envoy access logs 401 spikes.
**Mini example (YAML):**

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
spec:
  jwtRules:
  - issuer: "https://idp.example.com"
    jwksUri: "https://idp.example.com/.well-known/jwks.json"
```

**Response:** TheHive case; rotate client secrets; add malicious `aud`/`iss` patterns to detection.
**Standards:** Istio authn policy tasks. ([Istio][16])

---

### Use Case 20 — **Istio AuthorizationPolicy violations**

**Why:** Unauthorized service-to-service calls.
**Signals:** DENY logs hit; unexpected source principals.
**Mini example (YAML):**

```yaml
kind: AuthorizationPolicy
spec:
  action: DENY
  rules:
  - from:
    - source: { principals: ["cluster.local/ns/default/sa/unknown"] }
```

**Response:** Block, alert, update service account mappings; review RBAC.
**Standards:** Istio AuthorizationPolicy reference. ([Istio][17])

---

### Use Case 21 — **NGINX reverse-proxy hardening drift**

**Why:** Missing TLS, weak ciphers, open status endpoints.
**Signals:** Config change detected (Wazuh FIM); security headers missing in logs.
**Mini example:** NGINX hardening (TLS between proxy ↔ upstream, JWT/OIDC where applicable).
**Response:** Apply hardened config template; validate headers; regression tests.
**Standards:** NGINX security controls (official docs). ([NGINX Documentation][8])

---

### Use Case 22 — **Apache httpd risky modules or info leakage**

**Why:** `mod_status` exposure, directory listing.
**Signals:** Apache access logs `/server-status`; config include adds `Indexes`.
**Mini example:** Alert when `GET /server-status` without auth from external ASN.
**Response:** Disable or restrict `mod_status`, remove `Options Indexes`, enable TLS.
**Standards:** Apache security tips (2.4). ([Apache HTTP Server][18])

---

## Chapter E — Data, Storage & Keys

### Use Case 23 — **S3 public access / ACL policy change**

**Why:** Market data or PII exposure.
**Signals:** CloudTrail `PutBucketAcl|PutBucketPolicy` → public; Elastic alert.
**Mini example (KQL):**

```
event.dataset:"aws.cloudtrail" and event.action:("PutBucketAcl","PutBucketPolicy")
and aws.cloudtrail.request_parameters.policy:*"Principal":"*"
```

**Response:** Auto-block public ACLs, apply bucket policy guardrail, notify DPO/legal.
**Standards:** AWS Security Pillar; Security Hub FSBP standard. ([AWS Documentation][7])

---

### Use Case 24 — **KMS key disable / schedule deletion**

**Why:** Ransom or sabotage attempt.
**Signals:** CloudTrail `DisableKey|ScheduleKeyDeletion`.
**Mini example:** Alert if `PendingWindowInDays` < 7 for critical CMKs.
**Response:** Cancel deletion, rotate affected secrets, verify encryption posture.
**Standards:** AWS Well-Architected Security Pillar. ([AWS Documentation][7])

---

### Use Case 25 — **Public RDS snapshot sharing**

**Why:** Database snapshot data leak.
**Signals:** CloudTrail `ModifyDBSnapshotAttribute` to `restore` with `all`.
**Mini example (KQL):**

```
event.dataset:"aws.cloudtrail" and event.action:"ModifyDBSnapshotAttribute"
and aws.cloudtrail.request_parameters.attributeName:"restore" 
and aws.cloudtrail.request_parameters.valuesToAdd:"all"
```

**Response:** Revoke share, rotate DB creds, review snapshot policies.
**Standards:** CIS Controls v8 (Data Protection). ([CIS][19])

---

### Use Case 26 — **Bulk data exfiltration pattern**

**Why:** Sudden large `GetObject` or egress spikes.
**Signals:** CloudTrail S3 `GetObject` volumetrics + VPC Flow Logs high egress.
**Mini example:** Correlate `GetObject` count per principal with VPC bytes\_out > baseline + 3σ in 1h.
**Response:** Quarantine principal, block egress, start IR (preserve logs).
**Standards:** NIST CSF Detect/Respond; CloudTrail data events. ([NIST Publications][1])

---

## Chapter F — Threat Intel, Cases & Response Automation

### Use Case 27 — **IOC auto-enrichment with MISP via Cortex**

**Why:** Speed triage with context (sightings, tags, TLP).
**Signals:** Any alert with IP/Hash/Domain → Cortex MISP analyzer.
**Mini example:** TheHive automation: on new alert, run Cortex `MISP` analyzer; attach taxonomy & related events.
**Response:** If IOC is “known bad,” auto-block at NGINX / Istio egress; add to detection lists.
**Standards:** TheHive/Cortex MISP integration docs; MISP documentation. ([StrangeBee Docs][20])

---

### Use Case 28 — **Phishing domain feed → proactive blocks**

**Why:** Protect staff/partners portals.
**Signals:** MISP feed (phishing domains) → distribute to NGINX `map`/deny lists & DNS sinkhole.
**Mini example:** Cron job pulls MISP event tags “phishing” → render NGINX snippet of `server_name` deny rules.
**Response:** Announce to users; monitor hit counts in Elastic.
**Standards:** MISP project overview. ([MISP Threat Intelligence Platform][21])

---

### Use Case 29 — **Credential exposure (AWS keys on Git/host)**

**Why:** Immediate compromise risk.
**Signals:** Elastic detects keys in logs/code; GuardDuty “CredentialAccess\:IAMUser/AnomalousBehavior”.
**Mini example:** TheHive playbook: disable key, rotate, search last-used services; retro hunt in CloudTrail 7/30 days.
**Response:** Postmortem; add pre-commit secret scans; awareness training.
**Standards:** AWS GuardDuty + NIST 800-61 IR. ([Wazuh Documentation][4])

---

### Use Case 30 — **ATT\&CK coverage & purple-team drills**

**Why:** Ensure detections map to adversary TTPs used against exchanges.
**Signals:** Elastic Security “MITRE ATT\&CK coverage” view; prebuilt rules.
**Mini example:** Simulate `DeleteFlowLogs` (ATT\&CK Defense Evasion) and confirm alert firing; track coverage gaps.
**Response:** Add rules, test quarterly; report to governance.
**Standards:** MITRE ATT\&CK matrices; Elastic ATT\&CK coverage docs. ([MITRE ATT\&CK][22])

---

## Implementation Notes (Open-Source Stack Glue)

* **Ingest & normalize:** Use Elastic Agent/Filebeat AWS module for CloudTrail/VPC/ELB, plus syslog/JSON beats for NGINX/Apache/Istio Envoy. Wazuh ships endpoint logs and SCA/FIM/Rootcheck alerts to Elastic. ([Elastic][23])
* **Case mgmt & SOAR lite:** Alerts → TheHive cases; automations call **Cortex** for MISP, WHOIS, GeoIP, VT, etc. ([StrangeBee Docs][24])
* **Threat intel:** Operate your own **MISP**; tag feeds (TLP, galaxy clusters), push curated IOCs to blocklists. ([MISP Threat Intelligence Platform][21])
* **AWS posture:** Turn on **Security Hub** “Foundational Best Practices” for guardrails and evidence; map to SOC KPIs. ([AWS Documentation][25])

---

## Standards & Guidance (quick map)

* **NIST CSF 2.0** (functions Govern/Identify/Protect/Detect/Respond/Recover) for program alignment. ([NIST Publications][1])
* **NIST SP 800-53 Rev.5** for control families (AU, SI, AC, CM, IR). ([NIST Computer Security Resource Center][26])
* **NIST SP 800-61 Rev.3** for incident handling lifecycle. ([NIST Publications][5])
* **PCI DSS 4.0** (especially Req. 10 logging/monitoring, Req. 11 testing) for financial environments. ([Middlebury][27])
* **ISO/IEC 27001:2022 Annex A (e.g., 8.16 Monitoring)** and **ISO/IEC 27002:2022** implementation guidance. ([ISO][28])
* **CIS Controls v8/8.1** for prioritized safeguards; AWS guidance mappings. ([CIS][19])
* **AWS Well-Architected Security Pillar** & **Foundational Security Best Practices** for cloud-native guardrails. ([AWS Documentation][7])
* **Istio security best practices** (mTLS STRICT, authZ/authN). ([Istio][15])
* **NGINX/Apache hardening** references. ([NGINX Documentation][8])

---

### Bonus: TheHive case & Cortex enrichment (minimal example)

```json
{
  "title": "AWS Console login without MFA",
  "severity": 3,
  "tlp": 2,
  "tags": ["AWS","CloudTrail","Identity","NoMFA"],
  "description": "Elastic alert from CloudTrail: ConsoleLogin without MFA for IAM user X from IP Y",
  "customFields": {
    "principal": "arn:aws:iam::123456789012:user/trader01",
    "source_ip": "203.0.113.10"
  },
  "tasks": [
    { "title": "Lock account & require MFA" },
    { "title": "Cortex:MISP lookup on source_ip" },
    { "title": "CloudTrail pivot 24h; attach findings" }
  ]
}
```


[1]: https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf?utm_source=chatgpt.com "The NIST Cybersecurity Framework (CSF) 2.0"
[2]: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf?utm_source=chatgpt.com "NIST.SP.800-53r5.pdf"
[3]: https://www.nist.gov/cyberframework?utm_source=chatgpt.com "Cybersecurity Framework | NIST"
[4]: https://documentation.wazuh.com/current/cloud-security/amazon/services/supported-services/guardduty.html?utm_source=chatgpt.com "Amazon GuardDuty - Supported services"
[5]: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r3.pdf?utm_source=chatgpt.com "NIST.SP.800-61r3.pdf"
[6]: https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html?utm_source=chatgpt.com "Logging IP traffic using VPC Flow Logs"
[7]: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html?utm_source=chatgpt.com "AWS Well-Architected Framework - Security Pillar"
[8]: https://docs.nginx.com/nginx/admin-guide/security-controls/?utm_source=chatgpt.com "Security Controls | NGINX Documentation"
[9]: https://detection.fyi/elastic/detection-rules/integrations/aws/defense_evasion_ec2_flow_log_deletion/?utm_source=chatgpt.com "AWS VPC Flow Logs Deletion"
[10]: https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/index.html?utm_source=chatgpt.com "File integrity monitoring - Capabilities"
[11]: https://documentation.wazuh.com/current/user-manual/capabilities/malware-detection/rootkits-behavior-detection.html?utm_source=chatgpt.com "Rootkits behavior detection - Malware detection"
[12]: https://documentation.wazuh.com/current/proof-of-concept-guide/poc-file-integrity-monitoring.html?utm_source=chatgpt.com "File integrity monitoring - Proof of Concept guide"
[13]: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/rootcheck.html?utm_source=chatgpt.com "rootcheck - Local configuration (ossec.conf)"
[14]: https://documentation.wazuh.com/current/getting-started/use-cases/configuration-assessment.html?utm_source=chatgpt.com "Configuration assessment - Use cases"
[15]: https://istio.io/latest/docs/ops/best-practices/security/?utm_source=chatgpt.com "Security Best Practices"
[16]: https://istio.io/latest/docs/tasks/security/authentication/authn-policy/?utm_source=chatgpt.com "Authentication Policy"
[17]: https://istio.io/latest/docs/reference/config/security/authorization-policy/?utm_source=chatgpt.com "Authorization Policy"
[18]: https://httpd.apache.org/docs/2.4/misc/security_tips.html?utm_source=chatgpt.com "Security Tips - Apache HTTP Server Version 2.4"
[19]: https://www.cisecurity.org/controls/v8?utm_source=chatgpt.com "CIS Critical Security Controls Version 8"
[20]: https://docs.strangebee.com/cortex/?utm_source=chatgpt.com "Cortex - TheHive 5 Documentation"
[21]: https://www.misp-project.org/?utm_source=chatgpt.com "MISP Open Source Threat Intelligence Platform & Open ..."
[22]: https://attack.mitre.org/matrices/?utm_source=chatgpt.com "Enterprise Matrix"
[23]: https://www.elastic.co/guide/en/beats/filebeat/8.19/filebeat-module-aws.html?utm_source=chatgpt.com "AWS module | Filebeat Reference [8.19]"
[24]: https://docs.strangebee.com/?utm_source=chatgpt.com "TheHive 5 Documentation"
[25]: https://docs.aws.amazon.com/securityhub/latest/userguide/fsbp-standard.html?utm_source=chatgpt.com "AWS Foundational Security Best Practices standard in ..."
[26]: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final?utm_source=chatgpt.com "NIST SP 800-53 Rev. 5, \"Security and Privacy Controls for ..."
[27]: https://www.middlebury.edu/sites/default/files/2025-01/PCI-DSS-v4_0_1.pdf?fv=AKHVQBp6&utm_source=chatgpt.com "PCI-DSS-v4_0_1.pdf"
[28]: https://www.iso.org/standard/27001?utm_source=chatgpt.com "ISO/IEC 27001:2022 - Information security management ..."
