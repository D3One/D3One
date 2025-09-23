# AppLocker/WDAC Demo — Simulated Log Pack (Text-Only)

**This repository contains synthetic, text-based samples** meant to help blue teams test queries, detections, and analyst playbooks around AppLocker/WDAC and Windows process creation. These are **NOT real EVTX binaries** and **NOT exploits**. The goal is to visualize healthy telemetry and correlation patterns in a kiosk/ATM-like environment.

> Everything inside uses fictional hostnames, paths, hashes, SIDs, GUIDs, and IPs.

## Files

- `applocker_demo.evtx` — **text-based** pseudo-EVTX with AppLocker events (IDs 8003, 8004, 8007; plus a packaged app example).  
- `security_4688_demo.evtx` — **text-based** pseudo-EVTX with Windows Security **4688** (Process Creation) samples.  
- `readme.md` — this documentation with quick commands and field descriptions.

If you need JSON/CSV for SIEM ingestion, generate them from these text samples or ask your assistant to export pre-baked CSV/JSON variants.

## What these are / are not

- ✅ Safe for public training, tabletop exercises, dashboards, and demo pipelines.  
- ✅ Useful to validate **queries**, **parsers**, **detections**, and **enrichment** (hash, signer, user, parent/child).  
- ❌ Not executable, not loadable in Event Viewer as EVTX; they’re plain text for easy inspection.  
- ❌ Not a how-to or exploit guide.

## Event IDs & Meaning (cheat sheet)

- **AppLocker 8003** — Audit-only: *“would have been blocked”*. Use this to harden rules before switching to Enforce.  
- **AppLocker 8004** — Enforced block on EXE/DLL/Packaged app collections.  
- **AppLocker 8007** — Enforced block on MSI/Script collection.  
- **Security 4688** — Process Creation (enable *“Include command line in process creation events”* in GPO).  
- **WDAC/App Control** (if present) — look for `AppControlExecutableAudited` / `AppControlCodeIntegrityPolicyBlocked` in advanced hunting.

## Where to look (native logs)

```
Applications and Services Logs → Microsoft → Windows → AppLocker →
  (EXE and DLL | MSI and Script | Packaged app-*)
Security (Event ID 4688)
```

## Quick triage commands (read-only)

> These commands **only read** logs; they don’t change system state.

**PowerShell — last 50 AppLocker EXE/DLL blocks (8004):**
```powershell
Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" `
  -FilterXPath "*[System[(EventID=8004)]]" -MaxEvents 50 |
  Select-Object TimeCreated, Id, LevelDisplayName, Message
```

**PowerShell — AppLocker audit (8003) + MSI/Script blocks (8007):**
```powershell
Get-WinEvent -FilterHashtable @{
  LogName = @(
    "Microsoft-Windows-AppLocker/EXE and DLL",
    "Microsoft-Windows-AppLocker/MSI and Script"
  );
  Id = 8003,8007
} -MaxEvents 100 |
  Select TimeCreated, LogName, Id, Message
```

**wevtutil — quick export to text/EVTX:**
```cmd
wevtutil qe "Microsoft-Windows-AppLocker/EXE and DLL" /q:"*[System[(EventID=8004)]]" /c:20 /f:text
wevtutil epl "Microsoft-Windows-AppLocker/EXE and DLL" C:\Logs\applocker_exe_dll.evtx
```

**Security 4688 — include command lines:**
```powershell
Get-WinEvent -LogName Security -FilterXPath "*[System[(EventID=4688)]]" -MaxEvents 100 |
  Select TimeCreated, Id, @{n='Process';e={$_.Properties[5].Value}}, @{n='CmdLine';e={$_.Properties[8].Value}}
```

## Correlation workflow (analyst playbook)

1. **Start with blocks**: filter AppLocker **8004/8007** on kiosk/ATM endpoints.  
2. **Pivot to 4688** at the same timestamp → capture executable path, parent process, and command line.  
3. **Hash/signature enrichment** (EDR/Defender column): verify signer, publisher, and hash reputation.  
4. **Classify root cause**: overly broad **Path** rule, stale **Hash** allow-list, or loose **Publisher** scope.  
5. **Fix**: prefer **default-deny** + tight **Publisher** rules; automate allow-list refresh on updates; ensure service accounts are in scope.  
6. **Validate**: keep **8003** volume trending down; only then flip to Enforce.

## Replay-detection telemetry (safe illustration)

Even if traffic is encrypted, the money-moving channel must enforce **mutual auth**, **per-session keys**, **nonces/sequence**, and **hardware binding**. Example artifacts in the App file (fictional): *“Rejecting command: stale nonce … seq out-of-window.”* Build detections around *stale nonce*, *reused IV/seq*, or *device-identity mismatch* signals in application or middleware logs.

## Field notes

- There is no “change-the-hash” trick: a file’s hash is computed; change a byte and the old hash won’t match. Bypasses usually come from rule **precedence** (Path > Publisher gaps), trusted **container** processes, or stale allow-lists after updates.  
- Consider **WDAC** as your base (code integrity), then layer **AppLocker** for role scoping.  
- Kiosk UX matters: kill unintended file pickers, viewers, and hotkeys; gate maintenance behind physical keys + MFA.

---

**Author:** Ivan Piskunov — © 2018/2019, updated 2025  
This pack is a synthesis of lessons learned; it’s designed to help blue teams, not to enable exploitation.
