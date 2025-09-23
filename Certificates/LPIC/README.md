
# 🐧 LPIC-1 (2014 by I.P.) — Theory-&-Practice Q\&As for GNU/Linux Administrator (self eduction) 

**Author:** Ivan Piskunov
**Target vintage:** \~2014 (Ubuntu 14.04 LTS “Trusty”, Debian 7 “Wheezy”/early 8), SysV/Upstart heavy with some iproute2 tooling; content aligned to **LPIC-1 101/102** objectives of that era.

> This is an original practice set (not real exam items). It mirrors the **breadth** LPIC-1 expects: boot & runlevels, FHS, packaging (Deb/RPM), GNU/Unix commands, processes, networking, storage, permissions/ACLs, logging/scheduling, security basics. For official scope, see **LPI objectives**. ([Linux Professional Institute (LPI)][1])

---

## MAIN Q\&As (answers + quick “why”)

1. **Which directory holds host-specific, *variable* state like logs and spool?**
   **Answer:** `/var` (e.g., `/var/log`, `/var/spool`).
   **Why:** Per the **FHS 2.3**, “variable” data lives under `/var`; static host binaries live under `/usr`. ([pathname.com][2])

2. **What’s the difference between `/bin` vs `/sbin` (2014 view)?**
   **Answer:** User essentials vs. system admin essentials (root-oriented) needed in early boot and single-user mode.
   **Why:** FHS historic split; many modern distros later merged to `/usr/bin`, but LPIC-1 still tests the classic model. ([Linux Foundation Specs][3])

3. **How do you see the current runlevel on a SysV/Upstart box?**
   **Answer:** `runlevel` (prints previous/current), or check `/etc/inittab` on pure SysV.
   **Why:** LPIC-1 2014 still covers **SysV runlevels** (0–6, S). systemd later maps these to targets. ([man7.org][4])

4. **Change runlevel to multi-user (no GUI) immediately (SysV semantics):**
   **Answer:** `telinit 3`
   **Why:** `telinit` asks init to switch runlevels; on systemd hosts this is translated to the matching target. ([man7.org][5])

5. **Where do Debian packages come from and which low-level tool installs them?**
   **Answer:** `.deb` packages via **dpkg**; higher-level front ends are `apt-get`/`aptitude`.
   **Why:** `dpkg` is the medium-level tool; APT handles dependency resolution & repos. ([man7.org][6])

6. **Install a local `.deb` and fix missing deps (Debian/Ubuntu 2014):**
   **Answer:** `sudo dpkg -i pkg.deb && sudo apt-get -f install`
   **Why:** `dpkg` doesn’t resolve deps; `apt-get -f` repairs them.

7. **RPM land equivalent of “install package from repo and its deps”?**
   **Answer:** `sudo yum install <name>`
   **Why:** LPIC-1 is vendor-neutral; expect both APT and YUM/RPM basics.

8. **What file lists local users and their shells? What’s the field order?**
   **Answer:** `/etc/passwd`: `name:passwd:UID:GID:gecos:home:shell`
   **Why:** Classic Unix account database (shadowed password in `/etc/shadow` on modern systems).

9. **Create user with home, primary group, and Bash as login shell:**
   **Answer:** `sudo useradd -m -s /bin/bash -g finance jdoe`
   **Why:** `-m` makes home; `-s` sets shell; `-g` sets primary group.

10. **What do `SUID`, `SGID`, and the **sticky** bit do?**
    **Answer:** SUID runs with file owner’s UID; SGID with group’s GID (and for dirs, forces group inheritance); sticky on dirs limits deletes to owner/root.
    **Why:** Special permission bits alter execution/dir behavior; core LPIC-1 topic. Use `chmod 4755 file` for SUID, `chmod 2755 dir` for SGID, `chmod 1777 dir` for sticky. ([man7.org][7])

11. **Grant group *analytics* read on `/data/report.csv` without changing mode/owner:**
    **Answer:** `setfacl -m g:analytics:r /data/report.csv`
    **Why:** POSIX **ACLs** provide per-user/group perms beyond rwx triads. Verify with `getfacl`. ([Linux Documentation

    ][8])

12. **Find files >100MiB under `/var` modified in last 2 days:**
    **Answer:** `sudo find /var -type f -size +100M -mtime -2 -print`
    **Why:** `find` predicates combine size/time filters; staple admin skill.

13. **Compress a directory tree preserving perms/ownership:**
    **Answer:** `sudo tar -czpf backup.tgz /etc /var/www`
    **Why:** `-p` preserves perms; `-z` gzip; `-c` create; `-f` filename.

14. **Append command output and stderr to the same logfile:**
    **Answer:** `somecmd >> /var/log/tool.log 2>&1`
    **Why:** Redirections are processed left-to-right; `2>&1` attaches stderr to current stdout.

15. **View top CPU offenders and kill a process gently:**
    **Answer:** `top` (or `ps aux --sort=-%cpu | head`), then `kill -TERM <pid>`, escalate to `-KILL` if needed.
    **Why:** Signals: 15 graceful, 9 untrappable hard kill.

16. **Lower a job’s CPU priority when launching:**
    **Answer:** `nice -n 10 long_job`
    **Why:** Higher niceness → lower scheduling priority. Adjust live with `renice`.

17. **Show open TCP sockets and the owning PIDs (2014 tools):**
    **Answer:** `sudo netstat -tulpn` *(net-tools)* **or** `ss -tulpn` *(iproute2)*
    **Why:** LPIC-1 expects both families; `ss` is the modern replacement.

18. **Bring up an IP on `eth0` with iproute2:**
    **Answer:**

```
sudo ip addr add 192.0.2.10/24 dev eth0
sudo ip link set eth0 up
```

**Why:** `ip` is the consolidated tool for addr/links/route. ([man7.org][9])

19. **Add a default route and verify routing table:**
    **Answer:** `sudo ip route add default via 192.0.2.1` then `ip route`
    **Why:** iproute2 replaces `route`; check with `ip route show`. ([man7.org][10])

20. **Where do you set static IPs persistently on Debian vs. Ubuntu 14.04?**
    **Answer:** Debian 7: `/etc/network/interfaces`; Ubuntu 14.04 (Upstart era): also `/etc/network/interfaces` (no Netplan yet).
    **Why:** Netplan/systemd-networkd came much later; LPIC-1 2014 = ifupdown world.

21. **Resolve lookup order for hostnames before DNS?**
    **Answer:** `/etc/nsswitch.conf` line `hosts: files dns`
    **Why:** `nsswitch` controls resolver sources/priority.

22. **Which files influence hostname → IP lookups locally?**
    **Answer:** `/etc/hosts`, `/etc/resolv.conf` (for DNS servers).
    **Why:** Classic Unix resolver stack.

23. **Mount a new ext4 filesystem and make it persistent:**
    **Answer:**

```
sudo mkfs.ext4 /dev/sdb1
sudo mkdir -p /data
sudo mount /dev/sdb1 /data
echo '/dev/sdb1 /data ext4 defaults 0 2' | sudo tee -a /etc/fstab
```

**Why:** Correct fstab fields; `0 2` enables fsck (root fs should be `1`).

24. **Create 1 GiB swapfile (portable, 2014 style):**
    **Answer:**

```
sudo dd if=/dev/zero of=/swapfile bs=1M count=1024
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Why:** Mode 600 for safety; fstab line persists it.

25. **Rotate `/var/log/app.log` weekly, keep 8 copies, compress:**
    **Answer:** `/etc/logrotate.d/app`

```
/var/log/app.log {
  weekly
  rotate 8
  compress
  missingok
  notifempty
  create 0640 app app
  postrotate
    /bin/systemctl reload app.service 2>/dev/null || true
  endscript
}
```

**Why:** **logrotate** handles schedule, retention, compression, and post-hooks. (On pre-systemd, use init script reload.) ([Linux Documentation

][11])

26. **Schedule a script nightly at 01:00 for the current user:**
    **Answer:** `crontab -e` → `0 1 * * * /usr/local/bin/backup.sh`
    **Why:** `crontab(5)` describes the file format; `crontab(1)` manages it. ([man7.org][12])

27. **Archive `/srv` to a remote host over SSH (bandwidth-efficient):**
    **Answer:** `rsync -aHAX --delete /srv/ user@backup:/srv/`
    **Why:** `-a` preserves; `HAX` keeps hardlinks/ACLs/xattrs; push via SSH.

28. **Inspect kernel/runtime knobs for a process under `/proc`:**
    **Answer:** `/proc/<pid>/` (e.g., `status`, `cmdline`, `fd/`).
    **Why:** **procfs** exposes kernel data structures; mounted at `/proc`. ([man7.org][13])

29. **List loaded kernel modules and load/unload one:**
    **Answer:** `lsmod`; `sudo modprobe e1000e` / `sudo modprobe -r e1000e`
    **Why:** LPIC-1 wants you comfortable with module lifecycle.

30. **GRUB2: set a temporary kernel parameter at boot to enter single-user (rescue) mode:**
    **Answer:** In GRUB menu, press `e`, append `single` or `systemd.unit=rescue.target` to `linux` line, `Ctrl+X` to boot.
    **Why:** One-time kernel cmdline tweak; GRUB2 is the default bootloader. ([GNU][14])

31. **Make `/opt/tools` world-readable, only owner writable; add +x for dirs only:**
    **Answer:** `chmod -R u+rwX,go+rX,go-w /opt/tools`
    **Why:** Uppercase **X** sets execute only on directories (and existing exec files).

32. **Create a hard link and a symbolic link; when do they differ?**
    **Answer:** `ln fileA fileB` (hard), `ln -s fileA linkA` (soft).
    **Why:** Hard links share inode (same fs only); symlinks are path refs that can cross filesystems.

33. **Open SSH to a remote host with local port-forward:**
    **Answer:** `ssh -L 8080:localhost:80 admin@server`
    **Why:** Tunnels local 8080 → remote 80; staple admin technique.

34. **iptables: allow inbound SSH and drop everything else on INPUT (IPv4, quick lab):**
    **Answer:**

```
sudo iptables -P INPUT DROP
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

**Why:** Default-deny with stateful accept for existing flows; foundational LPIC-1 firewalling. ([Linux Documentation

][15])

35. **Why does LPIC-1 expect both `ifconfig/netstat` and `ip/ss` in 2014?**
    **Answer:** Because **net-tools** were still common while **iproute2** was the strategic replacement; admins had to be bilingual during the transition.
    **Why:** `ip(8)` subsumes addr/route/link; `ss` replaces netstat. ([man7.org][9])

---

## Compact cheat sheet (for 2014 year)

* **Boot & init:** BIOS/UEFI → GRUB2 → kernel → **SysV/Upstart** (systemd emerging). Runlevels (0–6, S) still matter for LPIC-1 phrasing. ([man7.org][4])
* **FHS:** `/etc` configs; `/usr` read-only shareable; `/var` variable; `/home` users; `/opt` add-ons; `/srv` service data. ([pathname.com][2])
* **Packaging:** Debian: `apt-get/apt, dpkg`; RPM: `yum, rpm`.
* **Users:** `/etc/passwd`, `/etc/shadow`, `useradd/usermod/passwd/chage`, groups with `groupadd/gpasswd`.
* **Permissions:** `chmod/chown/chgrp`, special bits SUID/SGID/sticky; **ACLs** via `getfacl/setfacl`. ([man7.org][16])
* **Processes:** `ps/top/pgrep/pkill`, signals (`kill -TERM/-KILL`), `nice/renice`, job control (`&`, `fg/bg`, `jobs`).
* **Networking:** `ip/ss` (modern) and `ifconfig/netstat` (legacy), `resolv.conf`, `nsswitch.conf`, `iptables`. ([man7.org][9])
* **Logs & schedules:** **rsyslog** + **logrotate**; `cron` (`crontab -e`, `crontab(5)` format) and `at`. ([Linux Documentation

  ][11])
* **Storage:** `fdisk/parted`, `mkfs`, `mount/umount`, `/etc/fstab`, `swap` setup.
* **Text ninja:** `grep/sed/awk/sort/uniq/cut/tr`, `less`, pipes/redirection.

---

## Sources & trusted prep (2014-friendly)

* **LPI LPIC-1 Objectives & overview** — official topics for Exams 101/102. ([Linux Professional Institute (LPI)][1])
* **FHS 2.3** — Filesystem Hierarchy Standard. ([pathname.com][2])
* **Ubuntu 14.04 docs** (period context for Upstart/ifupdown). ([files.ubuntu-manual.org][17])
* **Debian Administrator’s Handbook** — packaging, networking, services; includes APT vs aptitude notes. ([The Debian Administrator's Handbook][18])
* **Man pages (authoritative):**
  `ip(8)`/`ip-route(8)` (iproute2), `crontab(5)`/`crontab(1)`, `logrotate(8)`, `chmod(1)`, `getfacl(1)`. ([man7.org][9])

**Recommended books/web sites:**

* *LPIC-1 Linux Professional Institute Certification Study Guide* (exam 101/102; 2014–2015 editions). ([busindre.com][19])
* *UNIX and Linux System Administration Handbook* (4e, 2010) — timeless admin patterns.
* *The Debian Administrator’s Handbook* (free online). ([l.github.io][20])
* TLDP & man7.org for deep dives on classic tools and kernel bits. ([man7.org][13])

---

### Attribution
[1]: https://www.lpi.org/our-certifications/exam-101-102-objectives/?utm_source=chatgpt.com "LPIC-1 Exam 101 and 102 Objectives"
[2]: https://www.pathname.com/fhs/pub/fhs-2.3.pdf?utm_source=chatgpt.com "Filesystem Hierarchy Standard - Pathname Solutions"
[3]: https://refspecs.linuxfoundation.org/FHS_2.3/index.html?utm_source=chatgpt.com "FHS 2.3 Specifications"
[4]: https://www.man7.org/linux/man-pages/man8/runlevel.8.html?utm_source=chatgpt.com "runlevel(8) - Linux manual page"
[5]: https://www.man7.org/linux/man-pages/man8/telinit.8.html?utm_source=chatgpt.com "telinit(8) - Linux manual page"
[6]: https://man7.org/linux/man-pages/man1/dpkg.1.html?utm_source=chatgpt.com "dpkg(1) - Linux manual page"
[7]: https://man7.org/linux/man-pages/man1/chmod.1.html?utm_source=chatgpt.com "chmod(1) - Linux manual page"
[8]: https://linux.die.net/man/1/setfacl?utm_source=chatgpt.com "setfacl(1): set file access control lists - Linux man page"
[9]: https://www.man7.org/linux/man-pages/man8/ip.8.html?utm_source=chatgpt.com "ip(8) - Linux manual page"
[10]: https://www.man7.org/linux/man-pages/man8/ip-route.8.html?utm_source=chatgpt.com "ip-route(8) - Linux manual page"
[11]: https://linux.die.net/man/8/logrotate?utm_source=chatgpt.com "logrotate(8) - Linux man page"
[12]: https://man7.org/linux/man-pages/man5/crontab.5.html?utm_source=chatgpt.com "crontab(5) - Linux manual page"
[13]: https://man7.org/linux/man-pages/man5/proc.5.html?utm_source=chatgpt.com "proc(5) - Linux manual page"
[14]: https://www.gnu.org/software/grub/manual/grub/grub.html?utm_source=chatgpt.com "GNU GRUB Manual 2.12"
[15]: https://linux.die.net/man/8/iptables?utm_source=chatgpt.com "iptables(8) - Linux man page"
[16]: https://man7.org/linux/man-pages/man1/getfacl.1.html?utm_source=chatgpt.com "getfacl(1) - Linux manual page"
[17]: https://files.ubuntu-manual.org/manuals/getting-started-with-ubuntu/14.04e2/en_US/screen/Getting%20Started%20with%20Ubuntu%2014.04%20-%20Second%20edition.pdf?utm_source=chatgpt.com "Getting Started with Ubuntu 14.04 - files"
[18]: https://debian-handbook.info/browse/stable/?utm_source=chatgpt.com "The Debian Administrator's Handbook"
[19]: https://www.busindre.com/_media/lpic-1_linux_professional_institute_certification_study_guide_exam_101-400_and_exam_102-400_4th_edi.pdf?utm_source=chatgpt.com "LPIC-1 Linux Professional Institute Certification Study Guide"
[20]: https://l.github.io/debian-handbook/pdf/grayscale/en-US/debian-handbook.pdf?utm_source=chatgpt.com "The Debian Administrator's Handbook"
