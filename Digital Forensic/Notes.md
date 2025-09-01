
# **Digital Forensics: Technical Deep Dive with Open-Source Tools**

## **1. Introduction to Digital Forensics & Core Principles**

Digital Forensics involves the identification, preservation, analysis, and presentation of digital evidence. The process must adhere to strict principles to maintain evidence integrity:
- **Acquisition**: Creating a forensically sound bit-for-bit copy of data (e.g., using `dd` or `FTK Imager`).
- **Preservation**: Ensuring data remains unaltered via hashing (e.g., `SHA-256`) and write-blocking.
- **Analysis**: Examining data using specialized tools to extract artifacts and reconstruct events.
- **Reporting**: Documenting findings in a clear, reproducible manner for legal or investigative purposes.

This guide covers open-source tools, OS-specific artifacts, and technical workflows for Linux and Windows environments.

<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/f5dfae2f-9479-4717-9110-51ccb6b05337" />

---

## **2. Essential Open-Source Tools**

### **2.1 Memory Forensics with Volatility**
Volatility is the premier open-source framework for analyzing memory dumps. Key features:
- **Cross-OS Support**: Windows, Linux, macOS, Android.
- **Plugin Architecture**: Extensible via Python scripts.

**Installation (Volatility 3)**:
```bash
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
pip3 install -r requirements.txt
```

**Critical Commands**:
```bash
# Identify OS profile
python3 vol.py -f memory.dump windows.info

# List processes
python3 vol.py -f memory.dump windows.pslist

# Detect code injection (e.g., Cobalt Strike)
python3 vol.py -f memory.dump windows.malfind

# Extract hashes from memory
python3 vol.py -f memory.dump windows.hashdump

# Analyze network connections
python3 vol.py -f memory.dump windows.netscan
```

**Linux Memory Analysis**:
```bash
# Build Linux profile (requires kernel debugging symbols)
python3 vol.py -f linux_memory.dump linux.bash

# Recover bash history from memory
python3 vol.py -f linux_memory.dump linux.bash
```

### **2.2 Disk Forensics with The Sleuth Kit (TSK) & Autopsy**
TSK provides CLI tools for disk analysis, while Autopsy adds a GUI.

**Key TSK Commands**:
```bash
# Display partition structure
mmls disk_image.dd

# List files in a partition
fls -r -o 63 disk_image.dd

# Extract a deleted file by inode
icat -o 63 disk_image.dd 1285 > recovered_file.txt

# Calculate file hashes
sha256sum recovered_file.txt
```

**Autopsy Features**:
- Timeline analysis using `plaso`.
- Keyword searching and regex support.
- Integration with photo recognition tools (e.g., `Ghiro`).

### **2.3 Network Forensics with Wireshark & Xplico**
- **Wireshark**: Deep packet inspection and protocol analysis.
  - **Filters for IOC Detection**:
    ```bash
    http.request.method == "POST" and http contains "cmd.exe"
    tcp.flags.syn == 1 and tcp.flags.ack == 0  # SYN scans
    dns.qry.name contains "malicious.domain"
    ```
- **Xplico**: Network data reconstruction (e.g., HTTP, FTP, emails).
  ```bash
  xplico -f capture.pcap -o output_dir
  ```

### **2.4 Specialized Utilities**
- **Eric Zimmerman's Tools** (e.g., `RBCmd` for Registry analysis): Run on Linux via Wine and .NET 6:
  ```bash
  wget https://dotnet.microsoft.com/en-us/download/dotnet/6.0
  wine windowsdesktop-runtime-6.0.32-win-x64.exe
  ```
- **Log2timeline**: Create super-timelines from forensic artifacts.
- **Rekall**: Alternative to Volatility for memory analysis.

---

## **3. OS Architectures and Their Forensic Implications**

### **3.1 Windows Architecture & Artifacts**
Windows uses a **hybrid kernel** (NTOSKRNL.EXE) balancing performance and stability. Key components:
- **Registry**: Centralized database for system/user settings.
  - Critical hives: `SAM`, `SOFTWARE`, `SYSTEM`, `NTUSER.DAT`.
  - Tools: `RegRipper`, `FRED` (registry editor).
- **File Systems**: Primarily NTFS (journals, MFT, alternate data streams).
- **Artifact Locations**:
  - `C:\Windows\System32\winevt\Logs\`: Event logs (Security, System, Application).
  - `C:\Users\%USERNAME%\AppData\`: Application data (browsers, emails).
  - `C:\$MFT`: Master File Table (deleted files metadata).

**Example: Extracting USB History from Registry**
```bash
rip.pl -r SYSTEM -p usbstorage
```

### **3.2 Linux Architecture & Artifacts**
Linux uses a **monolithic kernel** where core services run in kernel space. Key differences:
- **No Registry**: Configurations scattered in `/etc/`, `~/.config/`, etc..
- **File Systems**: Ext4 (common), XFS (enterprise), Btrfs (snapshots).
- **Critical Artifacts**:
  - `~/.bash_history`: User command history (often targeted by attackers).
  - `/var/log/auth.log`: Authentication logs (success/failed logins).
  - `/var/log/syslog`: System-wide events.
  - `/etc/crontab`: Scheduled tasks.
  - `~/.ssh/known_hosts`: SSH connections history.

**Example: Analyzing auth.log for Bruteforce Attacks**
```bash
grep "Failed password" /var/log/auth.log | awk '{print $9}' | sort | uniq -c
```

---

## **4. Artifact Collection Techniques**

### **4.1 Live Data Acquisition on Linux**
**Volatile Data Collection Script**:
```bash
#!/bin/bash
# Live response script
output_dir="/opt/forensics/$(hostname)_$(date +%Y-%m-%d)"
mkdir -p $output_dir

# System info
date > $output_dir/date.txt
uname -a > $output_dir/uname.txt

# Processes & network
ps aux > $output_dir/ps_aux.txt
netstat -tulnp > $output_dir/netstat.txt
lsof > $output_dir/lsof.txt

# User activity
last > $output_dir/last.txt
cat ~/.bash_history > $output_dir/bash_history.txt

# Memory acquisition (requires LiME)
insmod lime.ko "path=$output_dir/memory.dump format=raw"
```
**Tools**: `LiME` (kernel module), `AVML` (user-space memory dumper).

### **4.2 Windows Artifact Collection**
- **Memory**: `Belkasoft RAM Capturer`, `Magnet RAM Capture`.
- **Disk Imaging**: `FTK Imager`, `Guymager` (Linux-based).
- **Registry Extraction**: `RegRipper`:
  ```bash
  rip.exe -r C:\Windows\System32\config\SYSTEM -p compname
  ```

### **4.3 Cloud & Container Forensics**
- **Docker**: Use `dof` (Docker Forensics Toolkit) to analyze container images.
- **Cloud**: `Turbinia` automates forensic workflows on GCP/AWS.

---

## **5. Practical Investigation Scenarios**

### **5.1 Case 1: Linux Web Server Compromise**
**Indicators**: Unauthorized SSH logins, cryptomining process.
**Investigation Steps**:
1.  **Check auth.log**:
    ```bash
    grep "Accepted password" /var/log/auth.log | tail -10
    ```
2.  **Analyze processes** (via memory dump):
    ```bash
    python3 vol.py -f memory.dump linux.pslist
    ```
3.  **Review cron jobs** for persistence:
    ```bash
    ls -la /etc/cron.d/ /var/spool/cron/
    ```
4.  **Extract malware** from memory:
    ```bash
    python3 vol.py -f memory.dump linux.procdump -D output_dir
    ```

### **5.2 Case 2: Windows Ransomware Incident**
**Indicators**: Encrypted files, ransom note, suspicious PowerShell.
**Investigation Steps**:
1.  **Analyze MFT** for file changes:
    ```bash
    mmls C:\image.dd
    fls -r -o 63 C:\image.dd
    ```
2.  **Extract ransomware execution from Event Logs**:
    ```bash
    rip.pl -r Security.evtx -p auditd
    ```
3.  **Check for lateral movement** via SSH keys or RDP logs:
    ```bash
    rip.pl -r SOFTWARE -p putty_sessions
    ```

---

## **6. Advanced Techniques & Tips**

### **6.1 Anti-Forensics Bypass**
- **Log Tampering**: Attackers often delete `/var/log/auth.log`. Recover from backups or memory.
- **File Hiding**: Use `extundelete` for Ext4 file recovery:
  ```bash
  extundelete /dev/sda1 --restore-file ~/.bash_history
  ```
- **Memory Injection**: Detect with `malfind` in Volatility.

### **6.2 Timeline Analysis**
Generate a super-timeline using `log2timeline`:
```bash
log2timeline.py --parsers "linux,apache" timeline.plaso disk_image.dd
psort.py -o l2tcsv timeline.plaso > timeline.csv
```

### **6.3 Integration with SIEM**
Forward logs to `Elasticsearch` or `Splunk` for correlation. Use `Elastic Security` for threat hunting.

---

## **7. Conclusion & Recommendations**

Linux and Windows forensics require different approaches due to architectural differences. Windows provides centralized evidence (Registry, Event Logs), while Linux artifacts are distributed across the file system. Key recommendations:
1.  **Prioritize Live Data**: Acquire memory and volatile artifacts first.
2.  **Use Multiple Tools**: Cross-validate findings with TSK, Volatility, and Autopsy.
3.  **Document Everything**: Maintain a chain of custody for legal compliance.
4.  **Stay Updated**: New anti-forensics techniques emerge regularly.

---

To be contuning...
