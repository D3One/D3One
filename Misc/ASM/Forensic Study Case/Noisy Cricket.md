

### ** Digital Forensic Challenge 2: "Operation Noisy Cricket: A Windows 7 Compromise"**

**Category:** Digital Forensics (Windows)
**Difficulty:** Medium-High
**Estimated Time:** 2 Hours
**Author:** D3One, Oleg usenko
**Date:** Circa 2015 (firt release at 2010)

#### **Challenge Description**

Hey, Mate.

We've received a concerned call from "Pretty Good Company, Inc." Their internal IDS flagged anomalous outbound traffic from the workstation of a senior accountant, `PC-JSMITH-WIN7`. The employee, John Smith, reported his system was "running slowly" yesterday afternoon.

The host was powered down and the drive was imaged. You have been provided with:
1.  A raw disk image: `pc-jsmith-win7.dd`
2.  A memory dump: `pc-jsmith-win7.mem`

Your mission is to analyze the evidence and answer the following questions:

1.  **Initial Vector:** What is the full path to the malicious document that initiated the attack?
2.  **Persistence:** What is the name of the scheduled task the malware created for persistence?
3.  **The Payload:** What is the name (not path) of the primary malicious executable that was dropped onto the system?
4.  **C&C:** What is the external Command & Control (C&C) IP address and port the malware was communicating with?
5.  **Data Theft:** What is the name of the important file that was recently accessed and likely exfiltrated from John's `Documents` folder?

**Tools at your disposal:** Autopsy (Sleuth Kit), Volatility, and any other command-line or open-source tools you see fit. Good luck.

---

### **Investigation Scenario & Step-by-Step Solution**

**Note for the Organizer:** The VM for this challenge should be based on **Windows 7 SP1 x86**. The following artifacts need to be planted on the system before imaging.

---

### **Phase 1: Disk Image Analysis with Autopsy**

We'll start with the disk image to get a general timeline and find obvious artifacts.

**Step 1.1: Timeline Analysis**
The first step is to create and analyze a timeline of file system activity. This can reveal what happened and when.

*   **Action:** In Autopsy, ingest the image and generate a timeline.
*   **Finding:** You sort the timeline by date and look for a flurry of activity around the time of the incident (`YYYY-MM-DD 16:00` UTC). You notice:
    *   A file `\Users\jsmith\Downloads\Q4_Earnings_Preview.scr` was accessed and executed.
    *   Shortly after, several files were created in `\Users\jsmith\AppData\Local\Temp\lowsec\`.
    *   A new scheduled task was created.
    *   A file `\Users\jsmith\Documents\project_merger_plan.pdf` was accessed shortly before the outbound network connection.

**Answer to Question 1:** The initial malicious file was `\Users\jsmith\Downloads\Q4_Earnings_Preview.scr`. (A screensaver file is a common lure).

**Answer to Question 5:** The file likely exfiltrated was `project_merger_plan.pdf`.

**Step 1.2: File System Browsing**
Now we hunt for the specific artifacts we saw in the timeline.

*   **Action:** Browse to `\Users\jsmith\AppData\Local\Temp\lowsec\`.
*   **Finding:** You find several files:
    *   `lsass.dll` (a attempt at hiding in plain sight)
    *   `svchosts.exe` (note the plural 's', a common trick)
    *   `config.ini`
*   **Action:** Examine `config.ini`.
*   **Finding:**
    ```ini
    [settings]
    c2_server=185.243.115.230
    c2_port=587
    sleep_time=300
    ```
**Answer to Question 4:** The C&C server is `185.243.115.230:587`. (Using port 587, often used for SMTP, is a trick to blend in with email traffic).

**Answer to Question 3:** The primary payload is `svchosts.exe`.

**Step 1.3: Persistence Mechanism**
*   **Action:** Browse to `\Windows\System32\Tasks` to see scheduled tasks.
*   **Finding:** Among the legitimate tasks (`GoogleUpdateTaskMachineUA`, etc.), you find a suspicious one: `\Windows\System32\Tasks\Microsoft\Windows\Multimedia\SystemSoundsService`.
*   **Action:** Examine the XML content of this task. It points to the action: `C:\Users\jsmith\AppData\Local\Temp\lowsec\svchosts.exe`.
**Answer to Question 2:** The scheduled task is `SystemSoundsService`.

---

### **Phase 2: Memory Analysis with Volatility**

Now let's confirm our findings and see the malware in action in memory.

**Step 2.1: Profile Identification**
First, we need to find the right Volatility profile for our memory dump.
*   **Command:**
    ```bash
    volatility -f pc-jsmith-win7.mem imageinfo
    ```
*   **Output:**
    ```
    Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86
    ...snip...
    ```

**Step 2.2: Process Analysis**
Let's look for suspicious processes. We know we're looking for `svchosts.exe`.
*   **Command:**
    ```bash
    volatility -f pc-jsmith-win7.mem --profile=Win7SP1x86_23418 pslist | grep -i svchost
    ```
*   **Output:**
    ```
    ...snip...
    0x857c0030 svchosts.exe            4024   2732      1       75      1      0 2024-04-09 16:02:18 UTC+0000
    ...snip...
    ```
    *Bingo!* The malicious process is running. Note its PID is `4024`.

**Step 2.3: Network Connections**
Let's see if we can find the network connection to the C&C server.
*   **Command:**
    ```bash
    volatility -f pc-jsmith-win7.mem --profile=Win7SP1x86_23418 netscan
    ```
*   **Output:**
    ```
    ...snip...
    TCPv4  192.168.1.15:49234   185.243.115.230:587   ESTABLISHED      4024    svchosts.exe
    ...snip...
    ```
    This confirms our finding from the disk analysis. The process `svchosts.exe` (PID 4024) has an active connection to the C&C server.

**Step 2.4: Dumping the Malicious Process**
For deeper analysis, we can dump the process from memory.
*   **Command:**
    ```bash
    volatility -f pc-jsmith-win7.mem --profile=Win7SP1x86_23418 memdump -p 4024 -D dump/
    ```
    This creates a file `4024.dmp` in the `dump/` folder. This can be analyzed with a tool like `strings` or uploaded to VirusTotal for further insight.

**Step 2.5: Checking for Rootkits (API Hooks)**
Sophisticated malware often hooks Windows APIs to hide itself. Let's check.
*   **Command:**
    ```bash
    volatility -f pc-jsmith-win7.mem --profile=Win7SP1x86_23418 apihooks -p 4024
    ```
*   **Output:**
    ```
    ...snip...
    Hook mode: Usermode
    Hook type: Inline/Trampoline
    Victim module: kernel32.dll (0x76ae0000 - 0x76c0d000)
    Function: kernel32.dll!CreateFileW at 0x76b0f7f0
    Hook address: 0xeb7000
    Hooking module: <unknown>
    ...snip...
    ```
    The output shows that the malware is indeed hooking API functions to hide its file and network activities.

---

### **Phase 3: Final Verification with CLI Tools (Simulated)**

If you had live access to the system, you would use these commands to confirm.

**Step 3.1: Scheduled Tasks**
*   **Command (CMD):**
    ```cmd
    schtasks /query /fo TABLE /v | find /i "sound"
    ```
*   **Output:**
    ```
    SystemSoundsService    C:\Users\jsmith\AppData\Local\Temp\lowsec\svchosts.exe  ...
    ```

**Step 3.2: Network Connections**
*   **Command (PowerShell):**
    ```powershell
    Get-NetTCPConnection | Where-Object {$_.State -eq 'Established'} | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess | Get-Process -Id { $_.OwningProcess } | Select-Object Name, Id, Path
    ```
*   **Output (simulated):**
    ```
    Name          Id Path
    ----          -- ----
    svchosts.exe 4024 C:\Users\jsmith\AppData\Local\Temp\lowsec\svchosts.exe
    ```

**Step 3.3: Recent Files**
*   **Command (CMD - looking at shell breadcrumbs):**
    ```cmd
    dir /a %USERPROFILE%\Recent\
    ```
*   **Finding:** You would likely see a shortcut file (`project_merger_plan.pdf.lnk`) confirming the user opened this document.

---

### **Summary of Answers**

1.  **Initial Vector:** `\Users\jsmith\Downloads\Q4_Earnings_Preview.scr`
2.  **Persistence:** `SystemSoundsService` (Scheduled Task)
3.  **The Payload:** `svchosts.exe`
4.  **C&C:** `185.243.115.230:587`
5.  **Data Theft:** `project_merger_plan.pdf`

This investigation guides the analyst through a classic attack chain: a phishing lure (scr file) leads to execution, which drops a payload that establishes persistence via a scheduled task and then communicates with a C&C server for data exfiltration. The solution leverages both disk and memory forensics to build a complete picture of the attack.
