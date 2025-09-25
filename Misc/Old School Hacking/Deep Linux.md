
# Deep Linux. Old-School Intrusion Playbooks (2000s)

> **Defender-first, historical lens.** This chapter reconstructs how Linux/UNIX servers were typically compromised in the early/mid/late 2000s. It emphasizes recognizable artifacts, misconfig patterns, and lab-safe checks so you can hunt and harden legacy estates—**not** run exploits.

## 1) Attacker Mental Model (then): Remote → Local → Pivot

Period guidance split intrusions into **remote access** (hit a listening service or other channel) and **local access** (once you’ve got a shell, escalate to root). The workflow was linear: exploit a network-facing daemon → land a shell → **priv-esc** → loot and pivot to adjacent systems. That framing shows up repeatedly in the era’s manuals and is still a great way to reason about Linux incidents. 

**Why it worked back then**

* Many daemons shipped **enabled by default** (NFS/RPC bits, FTP, r-services).
* Flat L2 segments and clear-text protocols made **sniffing** a gold mine.
* Host hardening was uneven; **SUID helpers**, world-writable dirs, and sloppy `sudoers` were common.
* Defenses existed (Tripwire/LIDS/St. Michael, syslog discipline), but were not universal.    

---

## 2) Recon & Service Mapping (what you’d still see in logs)

Attackers first built a service inventory, focusing on **RPC/NFS/NIS** and anything speaking old auth. Two lab-safe primitives that defenders can (and should) use when auditing legacy hosts:

```bash
# Enumerate RPC programs and ports
rpcinfo -p <target>
```

Sample era output (cut for brevity):

```
program vers proto port
100000   2    tcp  111  rpcbind
100005   1    udp  635  mountd
100003   2    udp  2049 nfs
100004   2    tcp  778  ypserv
```

This screams “NFS + NIS surface” and implies further probes (`showmount -e`, `rusers`, targeted port scans). 

Modern audit tip: scan RPC program maps with **Nmap RPC scan** to avoid guessing program numbers:

```bash
nmap -sS -sR <target>
```

Era guidance specifically highlighted `-sR` to enumerate RPC programs, making hand-rolled `rpcinfo -n ...` lookups unnecessary. 

---

## 3) Typical Remote Entry Points (and how to recognize them)

### 3.1 NFS / mountd / rpc.statd / portmap

**What went wrong (2000s):**

* `/etc/exports` with **wildcards**, broad subnets, and the dreaded `no_root_squash`.
* `portmap/rpcbind` exposed to the world; `mountd` reachable from untrusted networks.
* NFSv2/3 with UDP, no auth beyond client trust.

**What attackers looked for (your audit checklist):**

* `rpcinfo -p` showing `nfs`, `mountd`, `nlockmgr`. 
* `showmount -e <host>` returning wide-open exports (e.g., `/export *(rw,async,no_root_squash)`). 
* Tooling like **nfsshell** used to interact with exports once misconfig was found. 

**Defender moves (then and now):**

* Limit exports by **netgroup/host**, remove `no_root_squash`, prefer NFSv4 with strong auth.
* Filter 111/tcp&udp (rpcbind) and 2049 carefully; don’t expose NFS to the Internet.
* Log and alert on external `mountd` requests; they’re almost never legitimate.

### 3.2 The r-services: rsh / rlogin / rexec

**What went wrong:** Trust-based auth via `.rhosts`/`hosts.equiv` + no encryption. Enumerating who’s logged in or active (`rwho`, `rusers`) leaked operational details. 

**Artifacts defenders still find in legacy images:**

* `/home/*/.rhosts` with host/user trust chains.
* `inetd.conf` or `xinetd.d/` entries enabling `shell`, `login`, `exec`.
* Clear-text captures in old pcap archives showing rsh invocations.

**Mitigation:** disable r-services entirely; prefer SSH with keys and modern ciphersuites.

### 3.3 FTP daemons (wu-ftpd/ProFTPD/etc.)

**What went wrong:** Public FTP with **anonymous upload** or weak chroot; daemon bugs; uploads landing in webroot for “plant-and-execute”.

**Audit tells:**

* `nmap` finds 21/tcp; banner grabbing reveals ancient demon.
* World-writable upload dirs mapped under `/var/www/html/uploads`.
* Log entries with `STOR` to executable locations.

**Mitigation:** TLS-only, jailed sftp-subsystem, segregated uploads (`noexec,nodev,nosuid`).

### 3.4 BIND / DNS

**What went wrong:** Old **BIND** builds with remote overflows or logic bugs on authoritative servers; running as root; no chroot.

**Audit tells:** Named running as root (`ps`), writable zone files under world-writable dirs, ancient version banners.

**Mitigation:** chrooted named, least privilege, modern BIND/unbound, DNSSEC where appropriate.

### 3.5 Apache & web stacks (CGI/Perl/PHP)

The 2000s were peak era for **chunked-encoding parsing bugs**, canonicalization issues, and “**sample apps**” left enabled. Attacker flow: find a CGI or RFI/LFI on Apache/PHP → drop a web-backdoor → escalate locally. Era guidance lists canonicalization pitfalls, buffer overflows in web servers, and the importance of removing samples. 

**Scanning tools of the time:** **Nikto/Whisker** were staples for signature-based web misconfig/vulns. 

**Defender tells:**

* Access logs with odd query strings (`../../..`, `%2e%2e%2f`), `cmd=` parameters, or `?module=http://...` (RFI).
* Stray `.old`/`.bak` configs in webroot; `test.cgi`, `webcal/` samples.

---

## 4) Sniffing & MITM on Flat Networks

Pre-TLS-everywhere, **telnet**, **FTP**, POP/IMAP in clear text, and weak SSHv1 were common. Sniffers and MITM tools (dsniff, Ettercap, tcpdump) pulled credentials right off the wire, especially on hubs or poorly segmented switches. Linux-centric chapters catalog **dsniff, Ettercap, Sniffit, tcpdump**, and “switch sniffing” tricks that attackers and testers used. 

**Defender countermeasures (timeless):** eliminate legacy clear-text protocols; 802.1X; DHCP snooping/DAI; TLS everywhere; monitor for unexpected ARP floods.

---

## 5) Local Priv-Esc (the bridge from “uid=1000” to “uid=0”)

Once an attacker had any shell, the next step was **root**. The greatest hits of the era weren’t just kernel 2.4/2.6 bugs; a shocking amount of compromise came from *configuration*:

### 5.1 SUID/SGID and PATH/LD_* hijacks

**The pattern:** A root-owned helper with the SUID bit executes another program by *name* (not absolute path), or trusts `LD_LIBRARY_PATH`/`LD_PRELOAD` set in a root script—resulting in privilege escalation via **PATH hijack** or rogue shared libraries. The Linux volumes list SUID/SGID topic areas and shared library pitfalls as perennial sources of local root.  

**Blue-team checks:**

```bash
# Inventory SUID/SGID
find / -xdev -type f -perm -4000 -o -perm -2000 2>/dev/null
# Grep for unsafe constructs in root scripts
grep -R 'LD_LIBRARY_PATH\|/bin/sh\|-exec ' /etc /usr/local 2>/dev/null
```

### 5.2 `sudoers` antipatterns

`NOPASSWD` on wildcards or devops convenience entries like:

```
%deploy  ALL=(ALL) NOPASSWD: /usr/bin/vim, /usr/bin/find, /usr/bin/tar, /bin/sh
```

Many of these allowed shell escape to root.

### 5.3 Kernel priv-esc (2.4/2.6 era)

Specific CVEs evolved year-to-year; the generic lesson from the period texts is to assume **recurring kernel bugs** and adopt compilation-hardening and minimized attack surface. Defensive threads in the Linux chapters speak to **LIDS/Openwall** and kernel-mode stealth discussions, setting the “arms race” context later made famous by LKM rootkits.  

---

## 6) Persistence & Stealth (pre-EDR playbook)

**Simple persistence (then):**

* `cron` `@reboot` entries, `rc.local`, `init`/`xinetd` services.
* Dropping a secondary SSH key into `~root/.ssh/authorized_keys` with a non-obvious comment.
* SUID-root shells stashed under dot-dirs (e.g., `/bin/.sh`, `/usr/sbin/.sshd`).

**Rootkits (LKM/UDK era):**
Admin playbooks shifted from “use known-good `/bin/ps`” to **detecting kernel-level tampering** because attackers moved below user-land tools. Linux chapters index **kernel rootkits**, **LKM** usage, and defensive tools like **St. Michael** and **LIDS**; the general arms-race narrative (hook kernel APIs to blind `ps/ls/netstat`) is explicit in the series.   

**Blue-team heuristics (period-correct):**

* **Baseline hashes** of core binaries; verify from **read-only media** (CD) when in doubt.
* Tripwire/St. Michael/LIDS-style integrity; **boot from trusted media** for triage.
* Cross-view techniques (procfs vs. `ps`, pcap vs. `netstat`) to spot hiding. (The series describes why “known-good tools” alone became insufficient and why teams pushed checks **below** user-space.) 

---

## 7) Web-to-Root: The Classic Linux App Stack Kill Chain

1. **Find a web weakness** (RFI/LFI, bad upload, sample app).
2. **Drop a web backdoor** (often a single CGI/PHP that runs commands as the web user).
3. **Local recon & privesc** (SUID helpers, kernel bugs, sudoers missteps).
4. **Pivot** via SSH tunnels or living-off-the-land tooling.

Era chapters on web servers/appsec enumerate **canonicalization**, **buffer overflows**, **sample apps**, and **SQLi/XSS** patterns; scanners like **Nikto** were the go-to to surface low-hanging fruit.   

**Defender artifacts to hunt:**

* Apache access log anomalies (`..`, encoded traversal, external URLs in params).
* Odd files with web-server UID ownership in `/tmp`, `/var/tmp`, or upload dirs.
* `sudo` logs showing editors/archivers run as root from web UID.

---

## 8) The NIS/NFS Two-Step (period favorite)

**Pattern:** NIS (yp) and NFS often co-existed in shops. Poor isolation + world-readable NIS maps + permissive exports created chains: enumerate accounts via NIS → mount an export via NFS → replace a script or drop a setuid helper → wait for root to run it.

**Why we know:** The Linux volumes map **NIS/NFS** across several sections and call out their security implications and tooling (`nfsshell`, `showmount`, RPC maps), framing them as critical audit items.  

**Mitigation (then and now):**

* Retire NIS; isolate NFS with auth and strict exports; mount with `noexec,nodev,nosuid`.
* Monitor for **NFS mounts from untrusted IPs**; alert on `mountd` from the Internet.

---

## 9) “Evidence You’d See” — Era-accurate Snippets

> Use these to recognize 2000s-style weaknesses during assessments. They’re **not** exploit steps.

**RPC/NFS surface:**

```
# What a noisy, legacy box looked like
$ rpcinfo -p target
program vers proto port
100000  2    tcp   111  rpcbind
100005  1    udp   635  mountd
100003  2    udp  2049  nfs
100004  2    tcp   778  ypserv
```

Interpretation: RPC mapper + mountd + NFS + NIS. High risk if Internet-facing. 

**Nmap RPC scan (period-correct):**

```
$ nmap -sS -sR 192.0.2.10
# ... enumerates RPC programs without guessing numbers ...
```



**NFS export check (blue-team):**

```
$ showmount -e target
Export list for target:
/export (rw,async,no_root_squash)
```

If you see this in the wild, treat it as a five-alarm fire. 

**Web stack sanity (banner grabs & SSH hints):**
The Linux chapters even show banner/context captures for Apache and OpenSSH and remind that these services had frequent vulns—**keep versions tight** and configs strict.  

**Sniffer presence (what teams used):** dsniff, Ettercap, Sniffit, tcpdump; the book lays out UNIX sniffer workflows and “switch sniffing” concepts. If your IR set contains these binaries in odd locations or sees ARP shenanigans, treat as compromise. 

---

## 10) Logging, Detection, and Triage (period practices that still help)

* **Syslog hygiene**: centralize, rotate, and watch for gaps/roll-overs aligned with suspicious activity. 
* **scanlogd / Logcheck**: simple, era-authentic alerting for port scans and anomalies—great for low-noise legacy enclaves. 
* **Integrity tools**: LIDS/St. Michael/Tripwire-style baselines detect file tampering otherwise hidden by user-land tools.  

**Kernel-mode stealth reality check:** Defenders learned that “known-good `/bin/ps` from CD” isn’t enough if kernel APIs are hooked; you must validate below user-space and compare multiple views. The series documents this **arms race** explicitly. 

---

## 11) Hardening Legacy Linux (2000s pain points neutralized)

1. **Turn off** r-services; SSH-only with keys; disable SSHv1 and weak ciphers.
2. **Audit NFS/RPC** exposure; v4 only where possible; never Internet-facing; no `no_root_squash`.
3. **Chroot/least-privilege** for BIND/daemons; no root-run webservers.
4. **SUID diet**: inventory and remove where feasible; constrain `sudoers`.
5. **Web app discipline**: kill sample apps; sanitize upload dirs (`noexec,nodev,nosuid`); WAF only as a last line, not a band-aid.
6. **Integrity + logs**: baseline, centralize, correlate; teach teams to spot the artifacts above.

---

## 12) Why this still matters

You’ll still encounter brown-field estates with **NFS/NIS ghosts, SUID time bombs, and flat L2**. Recognizing the 2000s kill chain—**recon → network service weak point → local root → pivot**—lets you anticipate where attackers will win and close those doors in a structured way. The era’s texts remain crisp on this structure and the specific Linux service classes to audit first.  

---

### Mini-Glossary (period-correct tools & terms you’ll see in artifacts)

* **rpcinfo / showmount / nfsshell** — RPC/NFS enumeration/manipulation.  
* **dsniff / Ettercap / tcpdump / Sniffit** — UNIX sniffers and MITM tools. 
* **Nikto/Whisker** — web scanner mainstays. 
* **LIDS / St. Michael** — host integrity/hardening approaches.  

---

## Appendix A — Lab-Safe Audit Commands (copy/paste for defenders)

```bash
# 1) RPC/NFS exposure
rpcinfo -p $HOST
showmount -e $HOST

# 2) SUID/SGID inventory (common privesc source)
find / -xdev -type f -perm -4000 -o -perm -2000 2>/dev/null | sort

# 3) Look for r-services and inetd/xinetd leftovers
grep -R '^(shell|login|exec)\s' /etc/inetd.conf /etc/xinetd.d 2>/dev/null

# 4) Weak sudoers patterns (wildcards, editors, archivers)
grep -nE 'NOPASSWD|vim|nano|tar|less|man|find|python|perl|sh|bash' /etc/sudoers /etc/sudoers.d/* 2>/dev/null

# 5) Web upload dirs mounted safely
mount | grep -E ' /(var/www|srv)/.* (.*noexec.*nosuid.*nodev.*)'

# 6) Sniffer sanity (unexpected promisc NICs)
ip -d link show | grep -i promisc || true
```

*(Each command is defensive; none exploits anything.)*

---

### Sources (for this Linux chapter)

* *Hacking Exposed – Linux* (RPC/NFS/NIS mapping; web server/app sections; sniffers; Linux rootkit/IDS references; syslog/logging).       
* *Hacking Exposed* series (general model of remote→local→pivot; integrity checking; kernel-level stealth arms race).  

---
