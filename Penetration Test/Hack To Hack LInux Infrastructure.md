# 🐧 The 10 Most Common Ways Linux/Unix & OSS Stacks

*You live in AppSec/DevSecOps/Product Security and don’t want to moonlight as a red teamer. This overview focuses on **how attackers actually get in** across Linux/Unix and popular open-source components—and the specific controls that move risk down fast, with just enough technical detail to be useful.*

---

## 1) 🔑 SSH Exposure: Brute Force, Leaked Keys & “regreSSHion” (OpenSSH RCE)

*Short version:* Internet-facing SSH with password auth, recycled keys, or agent/forwarding mistakes remains the easiest front door. 2024 added spice with **CVE-2024-6387 (“regreSSHion”)**, a race-condition RCE in sshd on glibc-based systems. *Identity is still the perimeter—even on bare metal.* ([Qualys][1])

**Reality check**
Attackers spray creds, try default users, harvest keys from leaked repos, and pivot via agent-forwarding or poorly isolated bastions. Where patching lagged, **regreSSHion** turned unauthenticated network access into potential root on some platforms. *Yes, that was as bad as it sounds.* ([Qualys][1])

*What to do (now)* *—* *Disable password auth,* *enforce* **SSH certificates/FIDO2** *for humans, lock down* `sshd_config`, *and restrict source IPs. Patch OpenSSH past vulnerable ranges; add connection rate-limits and fail2ban only as a backstop—not a strategy.* ([Qualys][1])

---

## 2) 🧬 Supply-Chain Compromise: From Libraries to Tooling (xz utils, anyone?)

*Short version:* Open-source is a superpower—and a supply chain. The **xz utils** backdoor (**CVE-2024-3094**) proved a determined maintainer can ship malicious code into distros, potentially impacting **SSH authentication paths** on affected builds. *Trust, but verify (and pin).* ([Red Hat][2])

**Reality check**
Adversaries also ride typosquats in PyPI/NPM, malicious build steps, and poisoned images. Your Linux hosts pull packages, images, and scripts all day—so the “first compromise” can arrive via `apt upgrade` as easily as a spear phish.

*What to do (now)* *—* *Tighten provenance:* **SBOMs, signature verification (Sigstore/cosign), pinned digests**, minimal base images, and **repo allow-lists** for package managers and CI. *Continuously diff what’s deployed from what’s approved—don’t just scan once.*

---

## 3) 🌐 Web Apps on Linux: Old Faithful (Log4Shell & friends)

*Short version:* Your Linux stack usually serves apps. That means **OWASP Top 10** issues and “forever vulns” like **Log4Shell (CVE-2021-44228)** still drive compromises years later—because long-tail components never quite die. ([OWASP Foundation][3])

**Reality check**
Attackers chain trivial web bugs to footholds, then privesc locally. CISA continues to list Log4Shell as routinely exploited; patch debt + transitive dependencies = endless blast radius. ([CISA][4])

*What to do (now)* *—* *Inventory and kill* **vulnerable components** *(Log4j, struts, templating engines); put public apps behind a WAF but never rely on it; instrument egress to catch callback-style exploits; make “**SBOM + reachability analysis**” a release gate.* ([CISA][5])

---

## 4) 🧯 Local Priv-Esc: sudo, Dirty Pipe & OverlayFS Greatest Hits

*Short version:* Once inside, Linux privesc is still about **misconfigured sudoers/SUID** and kernel vulns. **sudo Baron Samedit (CVE-2021-3156)** and **Dirty Pipe (CVE-2022-0847)** are the poster children for “one bug to root.” *Treat local code exec as halfway to domain compromise.* ([Qualys][6])

**Reality check**
User-to-root turns a minor web foothold into a cluster-wide incident—especially where local admin is casually handed to service users.

*What to do (now)* *—* *Continuously audit* `sudoers` *for* `NOPASSWD` *and forced commands; prune SUID/SGID; enable* **LSM** *(AppArmor/SELinux)* *profiles; and keep kernels fresh (use live-patching where possible).*

---

## 5) 📊 Exposed Datastores & Dashboards (Redis, Elasticsearch, Grafana)

*Short version:* “Just a lab Grafana” on the internet? **Path traversal in Grafana (CVE-2021-43798)** and *unauthenticated* **Redis** exposures remain evergreen initial access and lateral-movement trampolines. Attackers commonly drop cron jobs via Redis or pivot off dashboards to underlying hosts. ([Grafana Labs][7])

*What to do (now)* *—* *Never expose* **Redis/Elasticsearch/Prometheus/Grafana** *directly. Enforce auth/TLS, IP allow-lists, and network segmentation; for Redis, enable protected mode, ACLs, and command blocks; watch for odd file-write patterns.* ([Redis][8])

---

## 6) ☸️ Kubernetes Control-Plane & etcd Missteps

*Short version:* The fastest route to “own everything” is a misconfigured cluster: **open API servers/kubelets**, **over-permissive RBAC**, and **unencrypted Secrets in etcd**. One token and it’s game over. *Cluster defaults are friendly to developers—not to you.* ([U.S. Department of War][9])

**Reality check**
etcd holds cluster truth; compromise bypasses admission controls and RBAC. Secrets aren’t encrypted at rest by default; many orgs still mount them as env vars, magnifying leakage risk. ([U.S. Department of War][9])

*What to do (now)* *—* *Close public access to API/kubelets; adopt* **NSA/CISA hardening**; **encrypt Secrets at rest**, prefer external KMS (or CSI/KMS) and mount-only consumption; minimize service account tokens; gate egress; and audit RBAC for wildcard verbs.* ([U.S. Department of War][9])

---

## 7) 📦 Container Escapes & Daemon Sockets

*Short version:* Containers aren’t a security boundary. 2024’s **“Leaky Vessels”** class (runC/BuildKit) reminded everyone that one cleverly crafted image can **escape to the host**. Also, mounting the Docker socket into builds is still “root-me.” ([wiz.io][10])

*What to do (now)* *—* *Patch* **runC/BuildKit**; disable privileged containers; adopt **rootless** where feasible; and use **seccomp/AppArmor** defaults. *Never mount* `/var/run/docker.sock`. *Scan images pre-deploy and at runtime; enforce signed images (cosign) and policy (Kyverno/OPA Gatekeeper).*

---

## 8) 🛠️ CI/CD Abuse (GitLab/Jenkins/GitHub Actions)

*Short version:* Build systems are high-trust and often internet-reachable. **GitLab CVE-2023-7028** allowed password resets to unverified emails—an ATO gift if 2FA wasn’t enforced. Runners that process untrusted PRs leak secrets or pivot into your cloud. ([NVD][11])

*What to do (now)* *—* *Force* **MFA everywhere** *(especially CI/CD accounts); restrict runner contexts; gate secrets with short-lived OIDC, not long-lived tokens; and treat pipeline artifacts/logs as sensitive.*

---

## 9) ☁️ Cloud Metadata (169.254.169.254) via SSRF

*Short version:* A garden-variety SSRF against an app on a Linux VM can fetch **instance role credentials** from the cloud metadata service and leap straight to your control plane. **IMDSv2** mitigations help, but only if you actually require them. ([AWS Documentation][12])

**Reality check**
The **Capital One** breach is still the canonical example: SSRF → metadata → credentials → data exfil. Same pattern works in many stacks if you leave metadata wide open. ([Capital One][13])

*What to do (now)* *—* *Require* **IMDSv2** *(tokenized)*, *tighten hop limits, and add egress controls and SSRF protections; prefer cloud-native roles with granular policies over static keys.* ([AWS Documentation][14])

---

## 10) 🧑‍💻 Identity & Config Hygiene on Linux: PAM, sudoers, and “forever creds”

*Short version:* Weak PAM chains, password logins, shared SSH keys, and zombie service accounts keep doors open. Attackers don’t need a 0-day if your `sudoers` says `NOPASSWD: ALL`. This is the unglamorous, high-ROI fix list.

*What to do (now)* *—* *Mandate* **key-based or FIDO2** SSH, move humans to **SSH certificates**, rotate service keys, **ban password SSH** entirely, and audit local groups. *Log and alert on new authorized_keys additions and changes to sudoers.*

---

# 🧭 Blue-Team Quick Wins (Minimal Drama, Maximum Risk Drop)

* *Kill password SSH, patch OpenSSH beyond **CVE-2024-6387** ranges, and enforce IP-scoped access to bastions.* ([Cisco][15])
* *Adopt **NSA/CISA Kubernetes Hardening**: close public control plane, RBAC least-privilege, **encrypt Secrets** at rest.* ([U.S. Department of War][9])
* *Remove public exposure for **Redis/Elasticsearch/Grafana**; enforce TLS+auth and known-good plugin sets.* ([Redis][8])
* *Continuously patch kernel & userland for **sudo/Dirty Pipe** style privesc and prune SUID/SGID.* ([Qualys][6])
* *Treat supply chain as code: **SBOMs, signatures, pinned digests**, policy enforcement on image/dep intake.*
* *Force MFA on **GitLab/Jenkins/GitHub**, restrict runners, and move secrets to short-lived OIDC.* ([NVD][11])
* *Require **IMDSv2** and egress/metadata guardrails to choke SSRF blast radius.* ([AWS Documentation][12])

---

# 🗓️ 30/60/90 for a Microsoft-free (ish) Estate

**30 days**
*Patch OpenSSH and disable password SSH; require IMDSv2; remove public Redis/ES/Grafana; start SBOM generation on builds.* ([Cisco][15])

**60 days**
*Apply NSA/CISA K8s guidance; enable Secrets encryption at rest; inventory/prune SUID/SGID; lock down sudoers; enforce MFA for CI/CD.* ([U.S. Department of War][9])

**90 days**
*Implement signed images and admission controls (Kyverno/OPA), rootless/least-privilege containers, RBAC attestation, and image provenance verification.* ([wiz.io][10])

---

# 🧩 Threat-Informed Mapping (for your detections)

* *Linux privesc:* **T1068, T1548** (MITRE ATT&CK)
* *Credentials from configs/keys:* **T1552**
* *Container escape/daemon abuse:* **T1611, T1610**
* *Cloud credentials from metadata:* **T1552.005**
  *Use the **ATT&CK Linux matrix** to round out coverage and drive purple-team tests.* ([MITRE ATT&CK][16])

---

# 📚 References & Further Reading (EN & RU)

* **OpenSSH / regreSSHion (CVE-2024-6387):** Qualys advisory & analysis; Cisco PSIRT summary; Wiz overview. ([Qualys][1])
* **xz utils backdoor (CVE-2024-3094):** Red Hat analysis; Wiz and Unit 42 briefs. ([Red Hat][2])
* **Kubernetes security:** NSA/CISA *Kubernetes Hardening Guide*; OWASP Kubernetes Top 10; Kubernetes docs on Secrets and encryption at rest. ([U.S. Department of War][9])
* **Local privesc:** sudo *Baron Samedit* (CVE-2021-3156); Dirty Pipe (CVE-2022-0847). ([Qualys][6])
* **Web app risk:** OWASP Top 10 (2021) and CISA guidance / KEV for **Log4Shell**. ([OWASP Foundation][3])
* **Datastore/dashboard exposure:** Grafana CVE-2021-43798; Redis hardening docs. ([NVD][17])
* **CI/CD:** GitLab **CVE-2023-7028** advisory & NVD. ([about.gitlab.com][18])
* **Cloud metadata SSRF / IMDSv2:** AWS docs on enabling/forcing IMDSv2; background on the Capital One breach. ([AWS Documentation][12])
* **Frameworks & baselines:** **MITRE ATT&CK (Linux matrix)**; **CIS Benchmarks** for RHEL/Ubuntu. ([MITRE ATT&CK][16])
* **RU sources (trends & guidance):** Positive Technologies (атаки на разработчиков/угрозы), Kaspersky *Securelist* (vuln & exploit stats), Yandex Cloud K8s security notes. ([ptsecurity.com][19])

---

[1]: https://blog.qualys.com/vulnerabilities-threat-research/2024/07/01/regresshion-remote-unauthenticated-code-execution-vulnerability-in-openssh-server?utm_source=chatgpt.com "OpenSSH CVE-2024-6387 RCE Vulnerability"
[2]: https://www.redhat.com/en/blog/understanding-red-hats-response-xz-security-incident?utm_source=chatgpt.com "Understanding Red Hat's response to the XZ security incident"
[3]: https://owasp.org/Top10/?utm_source=chatgpt.com "OWASP Top 10:2021"
[4]: https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-215a?utm_source=chatgpt.com "2022 Top Routinely Exploited Vulnerabilities"
[5]: https://www.cisa.gov/news-events/news/apache-log4j-vulnerability-guidance?utm_source=chatgpt.com "Apache Log4j Vulnerability Guidance"
[6]: https://blog.qualys.com/vulnerabilities-threat-research/2021/01/26/cve-2021-3156-heap-based-buffer-overflow-in-sudo-baron-samedit?utm_source=chatgpt.com "Sudo Vulnerability CVE-2021-3156: Root Access Risk"
[7]: https://grafana.com/blog/2021/12/08/an-update-on-0day-cve-2021-43798-grafana-directory-traversal/?utm_source=chatgpt.com "An update on 0day CVE-2021-43798"
[8]: https://redis.io/docs/latest/operate/oss_and_stack/management/security/?utm_source=chatgpt.com "Redis security | Docs"
[9]: https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF?utm_source=chatgpt.com "Kubernetes Hardening Guide"
[10]: https://www.wiz.io/blog/leaky-vessels-container-escape-vulnerabilities?utm_source=chatgpt.com "Leaky Vessels: Deep Dive on Container Escape ..."
[11]: https://nvd.nist.gov/vuln/detail/cve-2023-7028?utm_source=chatgpt.com "CVE-2023-7028 Detail - NVD"
[12]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-metadata-transition-to-version-2.html?utm_source=chatgpt.com "Transition to using Instance Metadata Service Version 2"
[13]: https://www.capitalone.com/digital/facts2019/?utm_source=chatgpt.com "2019 Capital One Cyber Incident | What Happened"
[14]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html?utm_source=chatgpt.com "Use the Instance Metadata Service to access ..."
[15]: https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-openssh-rce-2024.html?utm_source=chatgpt.com "Remote Unauthenticated Code Execution Vulnerability in ..."
[16]: https://attack.mitre.org/matrices/enterprise/linux/?utm_source=chatgpt.com "Linux Matrix - Enterprise"
[17]: https://nvd.nist.gov/vuln/detail/cve-2021-43798?utm_source=chatgpt.com "cve-2021-43798 - NVD"
[18]: https://about.gitlab.com/releases/2024/01/11/critical-security-release-gitlab-16-7-2-released/?utm_source=chatgpt.com "GitLab Critical Security Release: 16.7.2, 16.6.4, 16.5.6"
[19]: https://ptsecurity.com/ru-ru/about/news/pt-attacks-on-developers-via-github-and-gitlab-have-reached-a-record-high-in-three-years/?utm_source=chatgpt.com "атаки на разработчиков через GitHub и GitLab достигли ..."

---
