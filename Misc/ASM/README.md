
# 🧪 x86 Assembly Shenanigans: The Forbidden Lore

> **Disclaimer**: This repository contains code samples for **educational and research purposes only**. The techniques demonstrated are outdated, primarily relevant to legacy systems (circa 2000-2010), and are effectively useless on modern hardware with DEP, SMAP, SMEP, HVCI, and other mitigations. Do not use this for malicious purposes. I am not responsible for any damage caused by the misuse of this information. You are responsible for your own actions. Knowledge is power, but power comes with responsibility.

Hey there, fellow low-level enthusiast! 👋

This repo is a collection of my old university projects and research notes from around 2008-2010. Back when "Ring -1" was a myth, and we thought Ring 0 was the ultimate power. We're diving deep into the arcane arts of x86 assembly, exploring some... let's say "creative"...
ways to interact with hardware and operating systems that were definitely not covered in the official manuals.

These samples demonstrate concepts that are often found in rootkits, bootkits, and other low-level malware of that era. Understanding how these attacks work is the first step in building effective defenses. It's like learning lockpicking to become a better locksmith.

## 🗂️ Sample List

1.  **[Ring 3 to Ring 0 Transition via Call Gate](./sample_1_call_gate.md)**: The classic quest for ultimate privilege. Escaping the userland jail.
2.  **[Hardware Interrupt Hijacking for COM Port Snooping](./sample_2_com_int.md)**: Old-school keylogging? Packet sniffing? Let's talk to the 16550 UART directly.
3.  **[Direct ATA Disk Controller Manipulation](./sample_3_ata_controller.md)**: Talking to the hard drive's brain, bypassing the OS entirely. Scary stuff.
4.  **[Windows Kernel-Mode Driver for Stealth Execution](./sample_4_kernel_driver.md)**: Because sometimes your code needs to live in the land where no antivirus dares to go.

## 🚀 How to Use (or rather, how not to brick your PC)

**Seriously, be careful.** The best way to run this code is in a tightly controlled virtualized environment like **VMware Workstation** or **VirtualBox** with snapshots enabled. Target OSes should be period-accurate: Windows XP, Windows 2000, or even MS-DOS. Modern Windows (10/11) will laugh at these techniques and bluescreen your VM for fun. You have been warned.

*   **Assembler**: NASM or TASM is your friend.
*   **Debugger**: Good ol' OllyDbg or WinDbg in local kernel debug mode.
*   **For Drivers**: Windows Driver Kit (WDK) for the correct time period.

## 🔗 Resources & Further Reading

*   **[Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)** - The Bible. Vol. 3A is your best friend.
*   **[OSDev Wiki](https://wiki.osdev.org/Main_Page)** - An incredible resource for all things low-level.
*   **[MSDN Documentation](https://learn.microsoft.com/en-us/previous-versions/)** - Legacy Windows API and kernel docs.

## 🧠 Why This Matters (The "So What?" Factor)

This isn't just about writing malware. This is about understanding the fundamental principles of computer security.
*   **Reverse Engineering**: You can't reverse a rootkit if you don't know how it works.
*   **Exploit Development**: Understanding the system's inner workings is key to finding its weaknesses.
*   **CTF & Wargames**: Challenges involving kernel pwn, hardware hacking, or legacy systems often rely on these concepts. Check out some old-school CTF tasks from [DEF CON](https://forum.defcon.org/node/238/view) or more modern hardware challenges on [CTFtime](https://ctftime.org/).
