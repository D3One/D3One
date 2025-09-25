

# Deep Windows. Old-School Intrusions on Win2K/XP/Server 2003 

> **Intent.** Reconstruct how Windows servers were compromised in the 2000s, show what artifacts you’d still find today, and give lab-safe checks to harden legacy estates.

## 1) Threat Model (circa 2000s): Enum → Pop → Pillage → Persist

The 2000s Windows playbook was relentlessly practical: enumerate **over SMB/NetBIOS and MSRPC** (often via null sessions), land initial access via **RPC/DCOM, LSASS, or IIS misconfig/bugs**, then **loot credentials** and laterally move using admin shares and the Service Control Manager. The literature of the time documents this loop explicitly and even supplies console snippets we still see preserved in IR folders. 

---

## 2) Recon & Enumeration: “Shake the SMB Tree”

### 2.1 Null sessions & SMB/NetBIOS trivia that mattered

Anonymous `IPC$` connections (“null sessions”) could unlock wide reads on older/loosely configured systems and **on Windows Server 2003 domain controllers** with legacy compatibility. Tools then mined users, groups, trusts, shares, policies, even bits of the Registry via allowed paths.   

**Lab tell (should be blocked in prod):**

```bat
C:\>net use \\TARGET\IPC$ "" /u:""
The command completed successfully.
```

If that “succeeds,” your anonymous access policy and named-pipe ACLs need work. 

**Registry exposure gotchas.** If `HKLM\System\CurrentControlSet\Control\SecurePipeServer\Winreg` isn’t locked down, or **AllowedPaths** is too permissive, remote reads (even with limited context) can leak startup points and more. Era guidance: audit `Winreg` and **AllowedPaths**, and use tools like **DumpSec** to self-test.  

### 2.2 Resource Kit & all-in-one enumerators

**local/global** (Resource Kit) were the “hello world” of AD hygiene checks on a 2003 DC: 

```
C:\>local administrators \\caesars
Administrator
Enterprise Admins
Domain Admins
backadmin

C:\>global "domain admins" \\caesars
Administrator
backadmin
```

**DumpSec** (ex-DumpACL) did share/user/policy/service reports over a null session; command-line batch exports were common:
`dumpsec /computer=\\caesars /rpt=usersonly /saveas=tsv /outfile=c:\temp\users.txt`.  

**enum** concentrated SMB/LSA/AD queries (including password policy), and its output changed dramatically once you fixed legacy AD compatibility (see below). 

### 2.3 “Pre-Windows 2000 Compatible Access” (the DC foot-gun)

Selecting **legacy compatibility** at `dcpromo` adds **Everyone** and **ANONYMOUS LOGON** to the special group that gates directory reads. Result: null sessions could list users/groups until you **remove those identities and reboot DCs**; run enum again and get “Access is denied.” The book shows the “before/after” vividly.   

---

## 3) Initial Access (Network-service Greatest Hits)

### 3.1 MSRPC/DCOM (MS03-026): the Blaster era

Windows advertised **MSRPC** on 135/TCP&UDP (plus 139/445/593, and even over HTTP via IIS). In July 2003, LSD’s DCOM overflow brought **“one-packet-to-SYSTEM”** shells and a wave of point-and-shoot tooling (e.g., **kaht2**). Defender guidance from the time: **block** UDP 135/137/138/445 and TCP 135/139/445/593 (+ CIS/RPC over HTTP), and **patch** immediately.  

Console captures from era tools show exactly what IR folks preserved: remote shell returning `NT AUTHORITY\SYSTEM` on success.  

### 3.2 LSASS (MS04-011): SYSTEM-level RCE & mass infection

The **LSASS** flaw in MS04-011 yielded SYSTEM shells and widespread destabilization; the series rates it high and pairs it with RPC/DCOM as top-priority patch and segmentation focus for 2000s fleets. 

### 3.3 IIS canonicalization (Unicode/double-decode) & `ASP::$DATA`

IIS 4/5 suffered canonicalization bugs that let traversal slip past path checks (e.g., overlong UTF-8 `%c0%af`) and **NTFS ADS** trickery to read ASP source (`file.asp::$DATA`). Defender tells: look for requests like
`/scripts/..%c0%af..%c0%af..%c0%af../winnt/system32/cmd.exe?/c+dir` and `::$DATA` probes in logs; deploy URL normalization (URLScan back then), remove sample handlers, and **minimize extensions**.  

---

## 4) Credential Access: “Keys to the Kingdom”

### 4.1 pwdump2/pwdump3e & SysKey reality check

Even with SysKey encrypting SAM/AD, **hashes were still retrievable** (the weak point is access, not crypto). **pwdump2** injected into **LSASS** to read hashes; **pwdump3e** ran **remotely** against a compromised host over 139/445. That’s why old IR folders contain `pwdump` outputs and TSVs of users.   

Representative era snippet (trimmed):

```
Remote C:\>pwdump2
Administrator:500:...:...
Guest:501:...:...
SUPPORT_388945a0:1001:...:...
```

The book notes newer pwdump automatically finds **LSASS PID**; remote variants require admin-equivalent rights and SMB reachability.  

When memory injection was blocked, operators **stole SAM+SYSTEM** (to recover SysKey) or cracked **password history** via specialized tools. 

### 4.2 LSA Secrets (service/web/FTP creds; cached logons)

The **LSA Secrets** trove (HKLM\SECURITY\Policy\Secrets) exposed **service account passwords (effectively plaintext), web/FTP creds, computer account secrets, cached logons**. Tooling (e.g., **lsadump2**, Cain & Abel) used LSASS injection like pwdump to extract them. The guidance bluntly calls this a critical design risk if an attacker reaches **Administrator/SYSTEM**.   

> Defender takeaway: **service accounts** that log on with **domain privileges** were the fastest path from one compromised host to the **entire domain**. Hunt and rotate. 

### 4.3 LM hash cracking & rainbow tables

Because **LM hashes are unsalted** and split into two 7-char halves, brute force and later **rainbow tables** chewed through them rapidly; the book even lists contemporaneous speeds (JtR, LC, Cain) and table sizes/success rates. Blue-team action: **disable LM storage** (“Do Not Store LAN Manager Hash Value on Next Password Change”), enforce NTLMv2, and kill legacy LM auth on DCs.   

Sniffing NTLM/LM exchanges + cracking was another route (SMB capture/ARP tricks); the series details why **LMCompatibilityLevel** matters. 

---

## 5) Lateral Movement (pre-EDR): Admin Shares & SCM

Post-loot, lateral was mostly **admin shares** (`\\host\ADMIN$`), **SCM** service creation, and scheduled tasks — the “PsExec-style” pattern defenders still hunt for in logs and EDR today. Credential reuse across **SQL/Exchange/SharePoint** and **service accounts** made this startlingly effective in large domains. 

---

## 6) Persistence & Stealth (then)

* **Services** with benign names; **Run/RunOnce** keys; `at`/`schtasks` jobs.
* Extra **IIS** accounts (`IUSR_`, `IWAM_`) abused where web stacks coexisted. 

Kernel-mode stealth **did** arrive; defenders studied **KLister/KPP/KMCS** era countermeasures and cross-view detection (raw driver/object lists vs. user-mode tools). Use trusted media for verification; don’t trust on-box binaries. 

---

## 7) Web-to-SYSTEM on Windows (IIS chain in one picture)

1. **Canonicalization/ADS/WebDAV** weakness → drop a web backdoor (IUSR/IWAM context).
2. Local recon → **privesc** (service misconfigs/token mishaps).
3. **Dump creds** → lateral via SMB/SCM.

**Hunting tells:** overlong UTF-8 traversal (`%c0%af`), **`::$DATA`** requests, odd IIS accounts in logs or ACLs. **Fix**: purge sample ISAPI/handlers, URLScan/normalization, and least privilege for IIS.  

---

## 8) What Your IR Folder Would Contain (era-authentic artifacts)

* `users.txt`, `shares.tsv`, `services.txt` exported by **DumpSec**; often created after a null session succeeded. 
* `pwdump.out` or screenshots of **pwdump2/3e** with `Administrator`/`Guest` lines. 
* `lsadump2` dumps showing service account secrets, cached logons. 
* IIS logs with traversal/ADS probes (`..%c0%af..`, `::$DATA`). 

---

## 9) Blue-Team Playbook (2000s pain points neutralized)

1. **Kill anonymous enumeration:** remove **Everyone/ANONYMOUS LOGON** from **Pre-Windows 2000 Compatible Access** and reboot DCs; restrict anonymous named pipes/Registry paths.  
2. **RPC/DCOM exposure:** filter 135/137/138/139/445/593 (and CIS/RPC-over-HTTP); segment; patch MS03-026 class vulns. 
3. **Patch LSASS & IIS era bugs:** MS04-011; canonicalization/Unicode/double-decode; ADS. Normalize URLs and remove legacy handlers/samples.  
4. **Credential hygiene:** disable **LM** storage; enforce NTLMv2; rotate service accounts; minimize domain-privileged services on member servers and workstations.  
5. **Detection habits:** baseline services/autoruns; look for anonymous `IPC$` attempts and sudden dump-tool artifacts; confirm on trusted media when kernel-mode stealth is suspected.  

---

## 10) Lab-Safe Audit Snippets (copy/paste for defenders)

> These detect risk; they do **not** exploit anything.

**Anonymous SMB exposure (should fail):**

```bat
C:\>net use \\TARGET\IPC$ "" /u:""
```

If success → tighten anonymous named pipes & group memberships. 

**DumpSec (ensure anonymous dumps are blocked):**

```bat
C:\>dumpsec /computer=\\TARGET /rpt=shares /saveas=tsv /outfile=c:\temp\shares.tsv
```

This should **not** run anonymously against hardened servers/DCs. 

**Registry exposure sanity:**

```bat
# Check Winreg lockdown and AllowedPaths on a server (local)
# (Use Group Policy baselines for fleet enforcement)
```

Harden `SecurePipeServer\Winreg` and keep **AllowedPaths empty or minimal**. 

**AD legacy compatibility (on DCs):**

* Verify **Pre-Windows 2000 Compatible Access** has **no** Everyone/Anonymous; reboot required to take effect. 

**IIS logs hunt:**

* Search for `..%c0%af..` and `::$DATA` in `cs-uri-stem/query`; purge sample ISAPI/handlers and deploy URL normalization.  

**LM/NTLM posture:**

* Enforce “**Do Not Store LAN Manager Hash Value on Next Password Change**” and raise **LMCompatibilityLevel**; monitor for legacy LM auth.  

---

## 11) Why this still matters

Brown-field estates routinely keep ghosts of this era: permissive **anonymous access**, lingering **LM**, domain-privileged **service accounts**, and monolithic **IIS** installs with long-forgotten handlers. Knowing the 2000s kill chain and the artifacts above lets you **hunt fast** and **fix decisively**—before some nostalgia-driven attacker does.

---


