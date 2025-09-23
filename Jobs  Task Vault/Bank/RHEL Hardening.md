
# 🔒 Mini-Guide to OS Hardening for RHEL & Oracle Linux (circa 2014–2015)

**Author:** Ivan Piskunov
**What this is:** a compact, practitioner-style hardening checklist for **RHEL 6/7** and **Oracle Linux 6/7**. It compiles vendor docs and classic references I used on the job. *No third-party agents; vendor-built tools only.*
*Note:* test every change in a staging clone first; some knobs trade security for compatibility/perf. YMMV.

---

## 🧭 Scope & Version Notes

* **RHEL 6 ↔ SysV init, iptables, ntpd;** **RHEL 7 ↔ systemd, firewalld, chrony.**
* **Oracle Linux (OL)** tracks RHEL features; OL adds **Ksplice** and **UEK** options (see OL security guide). *Where commands differ, I show both.* ([Red Hat Documentation][1])

---

## 🧰 Method: “Reduce Attack Surface, Raise the Bar, Prove It”

1. **Minimize** (packages, services), 2) **Fortify** (kernel, network, auth), 3) **Monitor** (audit, logs), 4) **Prove** (SCAP/CIS/STIG scans), 5) **Recover** (backups, boot controls).

*Before you touch anything:* snapshot/backup **/etc**, **/boot**, firewall rules, **sshd\_config**, **sysctl.conf**, and audit rules.

```bash
# quick config bundle
sudo tar czf /root/pre-hardening-$(date +%F).tgz /etc /boot \
  /var/spool/cron /root/.ssh /etc/sysctl.conf /etc/ssh/sshd_config \
  /etc/audit/ /etc/grub* /etc/firewalld /etc/sysconfig/iptables
```

*Comment:* **Break-glass** path is everything when you harden remotely.

---

## 📦 Patching & Package Integrity

* **Keep current:**

  * RHEL: `yum update` (+ `yum-plugin-security`; `yum update --security` for CVE-focus on 6/7).
  * OL: via **ULN**/YUM; consider **Ksplice** for rebootless kernel/OpenSSL updates where licensed.
    *Why:* closes vulns fast; Ksplice shrinks maintenance windows. *Risk:* service restarts, rare regressions. ([Oracle Documentation][2])
* **Enforce GPG:** `/etc/yum.conf`: `gpgcheck=1`, per-repo `gpgkey=...`. *Why:* blocks unsigned packages.
* **Verify system files periodically:** `rpm -Va` (report drifts). *Why:* early drift/malware detection.

---

## 🧱 SELinux (don’t disable—it’s your seatbelt)

* **Mode:** `getenforce` → set **Enforcing**:

  * RHEL/OL: edit `/etc/selinux/config` → `SELINUX=enforcing`; temporarily: `setenforce 1`.
    *Why:* MAC confines processes; reduces blast radius. *Watch:* allow policies for legit services (httpd, db).
* **Tuning:** `semanage fcontext`, `restorecon -Rv`, and use `audit2allow` to craft minimal policies.
  *Comment:* Disabling SELinux to “fix” an app is a red flag; fix labels/booleans instead. ([Red Hat Documentation][1])

---

## 🚧 Host Firewall: default-deny

* **RHEL 7/OL 7 (firewalld):**

  ```bash
  firewall-cmd --set-default-zone=drop
  firewall-cmd --permanent --add-service=ssh
  firewall-cmd --permanent --add-port=443/tcp
  firewall-cmd --reload
  ```
* **RHEL 6/OL 6 (iptables):**

  ```bash
  iptables -P INPUT DROP; iptables -P FORWARD DROP; iptables -P OUTPUT ACCEPT
  iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
  iptables -A INPUT -p tcp --dport 22 -j ACCEPT
  service iptables save && service iptables restart
  ```

*Why:* only expose what you serve. *Watch:* passive FTP/cluster ports; add rules explicitly. ([Red Hat Documentation][1])

---

## 🧹 Services & Daemons (kill the noise)

* **List & disable:**

  * RHEL 7/OL 7: `systemctl list-unit-files --type=service`, then `systemctl disable --now avahi-daemon cups rpcbind` (if unused).
  * RHEL 6/OL 6: `chkconfig --list`, then `chkconfig avahi-daemon off; service avahi-daemon stop`.
    *Why:* each daemon is an attack surface. *Watch:* RPC/NFS, smartcard, bluetooth—bank servers rarely need them. ([Red Hat Documentation][1])

---

## 🔑 Accounts, PAM & Password Policy

* **Lock root for SSH & force sudo:**

  * `/etc/ssh/sshd_config`: `PermitRootLogin no`; manage admins via **wheel** group + `visudo` (`%wheel ALL=(ALL) ALL`).
    *Why:* auditability; no shared root. *Watch:* keep a console **break-glass** root path.
* **Password quality (era-correct):**

  * **RHEL/OL 6:** in `/etc/pam.d/system-auth` use **pam\_cracklib** (or pam\_pwquality if backported). Example:
    `password requisite pam_cracklib.so try_first_pass retry=3 minlen=12 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1`
  * **RHEL/OL 7:** `/etc/security/pwquality.conf`: `minlen=12 dcredit=-1 ucredit=-1 lcredit=-1 ocredit=-1` and ensure
    `password requisite pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=` in PAM.
    *Why:* enforce complexity. *Watch:* service accounts—set `!` lock or key-based auth only.
* **Lockout brute force:**

  * **RHEL/OL 6:** `pam_tally2.so` in `system-auth`: `deny=5 unlock_time=900`.
  * **RHEL/OL 7:** `pam_faillock.so` in `system-auth` & `password-auth`: `deny=5 unlock_time=900`.
    *Why:* slows online guessing; *Watch:* won’t stop credential stuffing via app layer.
* **Aging & reuse:** `/etc/login.defs` — `PASS_MAX_DAYS 90`, `PASS_MIN_DAYS 7`, `PASS_WARN_AGE 14`; in pam\_unix add `remember=5`.
* **UMASK:** `/etc/profile` and `/etc/login.defs`: `UMASK 027`. *Why:* sane default perms.

*References:* RHEL/OL security guides cover PAM, pwquality, faillock/tally2 specifics. ([Red Hat Documentation][1])

---

## 🗝️ SSH Hardening (OpenSSH 5.x/6.x era)

Edit `/etc/ssh/sshd_config`, then `service sshd reload` (RHEL6) or `systemctl reload sshd` (RHEL7).

* **Protocol:** `Protocol 2` (*should already be default*).
* **Key-only (preferred):** `PasswordAuthentication no`, `ChallengeResponseAuthentication no`.
* **Ciphers (2014–2015 safe set):**

  ```
  Ciphers aes256-ctr,aes192-ctr,aes128-ctr
  MACs hmac-sha2-512,hmac-sha2-256,hmac-ripemd160
  KexAlgorithms diffie-hellman-group-exchange-sha256,ecdh-sha2-nistp256,ecdh-sha2-nistp384
  ```

  *Why:* avoid CBC/RC4; prefer CTR and SHA-2 MACs. *Watch:* older clients; check support with `ssh -Q cipher -Q mac -Q kex`.
* **Access control:** `AllowUsers <admin1> <admin2>` **or** `AllowGroups wheel`.
* **Hygiene:** `LoginGraceTime 30`, `MaxAuthTries 3`, `ClientAliveInterval 300`, `ClientAliveCountMax 2`, `X11Forwarding no`, `AllowTcpForwarding no`, `UseDNS no`, `Banner /etc/issue.net`.
  *Comment:* lock it down, but don’t lock yourself out—keep an out-of-band console.

*Reference baselines:* CIS/STIG profiles enumerate era-appropriate SSH knobs. ([CIS][3])

---

## 🧩 Kernel & TCP/IP (sysctl)

Append to `/etc/sysctl.conf` (then `sysctl -p`):

```
# IP hygiene
net.ipv4.ip_forward = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.rp_filter = 1    # use 2 (loose) if asymmetric routing
net.ipv4.conf.default.rp_filter = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_rfc1337 = 1           # mitigate TIME-WAIT assassination

# Kernel hardening
kernel.randomize_va_space = 2
kernel.kptr_restrict = 1
kernel.dmesg_restrict = 1
fs.suid_dumpable = 0
kernel.sysrq = 0
vm.mmap_min_addr = 65536
```

**Optional (case-by-case):**

* Disable IPv6 if truly unused: `net.ipv6.conf.all.disable_ipv6=1`. *Watch:* breaking modern stacks.
* Lock module loading post-boot: `kernel.modules_disabled=1` (*very aggressive; breaks DKMS, some drivers*).

*Why these help:* closes routing/redirect tricks, makes kernel info less leak-y, and reduces SUID core risks. *Trade-offs:* PMTU relies on some ICMP—don’t nuke all ICMP or you’ll get weird performance. *Refer to vendor guides for each knob’s semantics.* ([Red Hat Documentation][1])

---

## 📁 Filesystems, Mounts & Critical Perms

* **Harden temp areas:** in `/etc/fstab`

  ```
  tmpfs /tmp     tmpfs  defaults,noexec,nosuid,nodev  0 0
  tmpfs /dev/shm tmpfs  defaults,noexec,nosuid,nodev  0 0
  /tmp /var/tmp  none   bind
  ```

  *Why:* blocks script execution from temp and device files. *Watch:* some installers want `exec`—flip temporarily if needed.
* **Tighten perms:**

  * `/etc/shadow` `600 root:root`; `/etc/ssh/ssh_host_*_key` `600 root:root`; root’s `~/.ssh` `700`.
  * Set default UMASK to `027`.
* **Bootloader security:**

  * **RHEL/OL 7 (GRUB2):** `grub2-setpassword` (creates `/boot/grub2/user.cfg`).
  * **RHEL/OL 6 (GRUB legacy):** add `password --md5 <hash>` in `/boot/grub/grub.conf`.
    *Why:* prevents offline single-user tampering. *Watch:* document the password in your vault.
* **Single-user auth:** ensure `sulogin` prompts for root on rescue/emergency. On systemd, verify `sulogin.conf`.

*Refs:* Vendor docs detail GRUB and fstab options. ([Red Hat Documentation][1])

---

## 📝 Logging, Time & Auditing

* **Time sync:**

  * RHEL/OL 7: `chrony` (`systemctl enable --now chronyd`)
  * RHEL/OL 6: `ntpd` (restrict to trusted sources)
    *Why:* sane logs & Kerberos; *Watch:* NTP ACLs.
* **Rsyslog:** forward security logs to a remote collector; use `@@loghost:514` (TCP) with TLS if available.
* **Auditd:** enable and load sane rules:

  ```
  # identity and scope
  -w /etc/passwd -p wa -k identity
  -w /etc/shadow -p wa -k identity
  -w /etc/sudoers -p wa -k scope
  # privileged binaries
  -a always,exit -F path=/usr/bin/sudo -F perm=x -F auid>=1000 -F auid!=4294967295 -k priv_esc
  # time changes
  -w /etc/localtime -p wa -k time
  ```

  `augenrules --load` or `service auditd restart`.
  *Why:* tamper evidence & forensics. *Watch:* audit volume; tune keys.
* **Logrotate:** verify rotations for `/var/log/secure`, `/var/log/audit/audit.log`.

*Baselines:* RHEL/OL security guides + DISA STIG include canonical audit sets. ([Red Hat Documentation][1])

---

## 🧪 Integrity & Compliance (built-in tooling)

* **AIDE:** `yum install aide; aide --init; mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz` and cron a daily check.
  *Why:* detects file tampering. *Watch:* update DB after legit patching.
* **OpenSCAP (oscap):** scan against **CIS/STIG**/**RHEL/OL** profiles:

  ```bash
  yum install scap-security-guide openscap-scanner
  oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_stig \
    --results /root/scan-$(date +%F).xml /usr/share/xml/scap/ssg/content/ssg-rhel7-ds.xml
  ```

  *Why:* evidence-based compliance. *Watch:* remediation rules may be blunt—review first. ([static.open-scap.org][4])
* **FIPS mode (when required):** follow vendor steps (rebuild initramfs, kernel cmdline flags). *Why:* standardized crypto; *Watch:* legacy protocols may break.\* ([Oracle Documentation][5])

---

## 🧿 Oracle Linux-specific Tips

* **Ksplice** for zero-downtime kernel/userland critical patches where supported/licensed.
* **UEK vs RHCK:** UEK brings newer kernel features; validate drivers/perf under your workloads before switching.
  *Ref:* Oracle Linux security docs. ([Oracle Documentation][2])

---

## 🔄 Ops Hygiene & Backups

* **Baseline configs under version control** (even a private git repo on the box for `/etc` is fine in 2015 ops).
* **Cron job to back up deltas** of `/etc`, firewall, SSH, audit rules to a remote, root-only bucket or share.
* **Document** a “rollback playbook” for each change category (SSH, firewall, PAM) and store out-of-band.

---

## ✅ Recommended Baseline (TL;DR)

* **SELinux:** Enforcing; fix with `semanage`/booleans, don’t disable.
* **Firewall:** default-drop; allow only needed.
* **SSH:** key-only; CTR ciphers; SHA-2 MACs; root SSH disabled; allow-list admins.
* **PAM:** pwquality/cracklib; lockout 5/15; aging 90/7/14; UMASK 027.
* **sysctl:** redirect/rp\_filter/ICMP hygiene; ASLR=2; dmesg/kptr restrict; suid dump off.
* **FS/mounts:** noexec/nosuid/nodev on tmp/devshm; GRUB password; single-user requires auth.
* **Time & logs:** chrony/ntpd; remote syslog; auditd rules covering identity/privilege/time.
* **Updates & integrity:** timely patching; GPG on; AIDE; periodic OpenSCAP scans; (OL) Ksplice if available.

---

## 📎 Appendix: Era-correct SSH snippet (server)

```
# /etc/ssh/sshd_config (RHEL/OL 6/7 era)
Protocol 2
PermitRootLogin no
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
Ciphers aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512,hmac-sha2-256,hmac-ripemd160
KexAlgorithms diffie-hellman-group-exchange-sha256,ecdh-sha2-nistp256,ecdh-sha2-nistp384
LoginGraceTime 30
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
AllowGroups wheel
Banner /etc/issue.net
UseDNS no
```

*Comment:* verify available algos with `ssh -Q` on your specific OpenSSH build.

---

## 📚 Official Documentation & Solid References

* **RHEL 7 Security Guide** (vendor): end-to-end hardening topics (SELinux, firewalld, auditing, OpenSCAP). ([Red Hat Documentation][1])
* **RHEL 6 Security Guide** (vendor): SysV/iptables era specifics & PAM (cracklib/tally2). ([Red Hat Documentation][6])
* **Oracle Linux 7 Security Guide** (vendor) + FIPS steps. ([Oracle Documentation][2])
* **Oracle Linux 6 Security Guide** (vendor) and lifecycle note for OL6. ([Oracle Documentation][7])
* **DISA STIG** for RHEL/OL 7 (baseline controls), and Red Hat’s STIG portal. ([public.cyber.mil][8])
* **SCAP Security Guide (OpenSCAP)** for RHEL/OL, with CIS/STIG profiles. ([static.open-scap.org][4])
* **CIS Benchmarks** landing (RHEL). ([CIS][3])
* **NIST SP 800-123: Guide to General Server Security** (mindset + checklists). ([nvlpubs.nist.gov][9])

---

*If you want, I can convert this into a one-pager checklist (CSV/Markdown) or tailor it for a specific bank role profile (DMZ web, core DB, jump host) from that 2014–2015 stack.*

[1]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/security_guide/index?utm_source=chatgpt.com "Security Guide | Red Hat Enterprise Linux | 7"
[2]: https://docs.oracle.com/en/operating-systems/oracle-linux/7/security/?utm_source=chatgpt.com "Oracle Linux 7 Security Guide"
[3]: https://www.cisecurity.org/benchmark/red_hat_linux?utm_source=chatgpt.com "CIS Red Hat Enterprise Linux Benchmarks"
[4]: https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-index.html?utm_source=chatgpt.com "Guide to the Secure Configuration of Red Hat Enterprise ..."
[5]: https://docs.oracle.com/cd/E77565_01/security/OL7-SECURITY.pdf?utm_source=chatgpt.com "Security Guide"
[6]: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/security_guide/index?utm_source=chatgpt.com "Security Guide | Red Hat Enterprise Linux | 6"
[7]: https://docs.oracle.com/en/operating-systems/oracle-linux/6/security/?utm_source=chatgpt.com "Oracle® Linux 6 Security Guide"
[8]: https://public.cyber.mil/stigs/downloads/?utm_source=chatgpt.com "STIGs Document Library"
[9]: https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-123.pdf?utm_source=chatgpt.com "NIST SP 800-123, Guide to General Server Security"
