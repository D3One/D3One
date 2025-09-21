# Sample 3: Direct ATA Disk Controller Manipulation

**Category**: Hardware Hacking / Firmware Abuse
**Relevance Period**: ~1990s - Mid 2010s (ATA/IDE drives). Modern SATA drives use AHCI and have stricter security (e.g., SCT Command Transport), but some concepts translate to older embedded systems.

## Overview

Time to talk to the big boys: the Hard Disk Drive (HDD) itself. Forget the OS, forget the file system. We're going to speak the language of the disk controller directly over the **ATA (Advanced Technology Attachment)** bus. This is how you *really* piss off an forensics investigator.

Why would you do this?
*   **Bricking a drive**: Send a `SECURE ERASE` command. Poof. Data gone forever (if the drive supports it).
*   **Head parking**: Spam the `STANDBY IMMEDIATE` command. This is the equivalent of DDoSing the physical read/write heads. Can cause increased wear and tear.
*   **Weird sounds**: Manually control the actuator arm by seeking to random sectors (`SEEK` command). Enjoy the symphony of clicks and whirs. It's like playing the piano, but more expensive if it breaks.
*   **Data hiding**: Read from/write to sectors directly, completely bypassing the OS's file system. This is the classic home for **rootkits** that hide in the HDD's **Service Area** (negative sectors, outside user-addressable space). Famous rootkits like **TDL4** did exactly this.

The catch? You need **Ring 0** privileges (kernel mode) to talk directly to the I/O ports controlling the disk controller. So this sample often pairs with Sample 1.

## The Code (NASM-style ASM for DOS/Win9x era)

```asm
BITS 16
ORG 0x100
section .text

; This code is designed for a real-mode environment like DOS or the Windows 9x DOS box.
; It directly talks to the Primary ATA controller's I/O ports (typically 0x1F0-0x1F7).

start:
    mov ax, 0x07C0          ; Set up segment registers for a .COM file
    mov ds, ax

    ; Let's send a STANDBY IMMEDIATE command (0xE0) to the drive.
    ; This parks the heads and spins down the drive. Annoying, isn't it?
    mov dx, 0x1F6           ; Drive/Head port (Primary bus, Master drive)
    mov al, 0xA0            ; Master drive (0xA0 for master, 0xB0 for slave)
    out dx, al

    ; Wait for the drive to be ready (BSY clear, DRDY set)
    call ata_wait

    mov dx, 0x1F7           ; Command port
    mov al, 0xE0            ; STANDBY IMMEDIATE command
    out dx, al

    ; Wait for the command to complete
    call ata_wait

    ; Let's do something more fun: read a sector directly!
    ; We'll read the Master Boot Record (MBR) from LBA 0.
    mov si, read_buffer
    mov eax, 0              ; LBA 0 (the MBR)
    mov cl, 1               ; Sector count
    call ata_read_sectors

    ; Now the MBR is in our buffer at `read_buffer`.
    ; We could modify it here and write it back to install a bootkit.
    ; But let's just hang.
    cli
    hlt

; -------------------- ATA WAIT SUBROUTINE --------------------
ata_wait:
    mov dx, 0x1F7           ; Status register
.ata_wait_loop:
    in al, dx
    test al, 0x80           ; Test BSY bit
    jnz .ata_wait_loop      ; If BSY=1, keep waiting
    test al, 0x01           ; Test ERR bit
    jnz .ata_error          ; Jump if error
    ret
.ata_error:
    ; Handle error (not implemented for brevity)
    ret

; -------------------- ATA READ SECTORS --------------------
; Input: EAX = LBA, CL = sector count, DS:SI = buffer
ata_read_sectors:
    pusha

    ; Select drive and LBA high bits
    mov dx, 0x1F6           ; Drive/Head port
    mov bl, al              ; Save low 8 bits of LBA
    and bl, 0x0F            ; Keep only lower 4 bits for later
    or bl, 0x40             ; LBA mode (bit 6)
    or bl, 0xA0             ; Select Master drive (0xA0)
    mov al, bl
    out dx, al

    ; Send sector count
    mov dx, 0x1F2
    mov al, cl
    out dx, al

    ; Send LBA low, middle, high bytes
    mov dx, 0x1F3           ; LBA Low port
    mov al, bl              ; Get our saved LBA low bits
    out dx, al

    inc dx                  ; 0x1F4 - LBA Mid port
    shr eax, 8
    out dx, al

    inc dx                  ; 0x1F5 - LBA High port
    shr eax, 8
    out dx, al

    ; Send READ SECTORS command (0x20)
    mov dx, 0x1F7
    mov al, 0x20
    out dx, al

    ; Wait for DRQ (Data Request) to be set
.read_wait:
    in al, dx
    test al, 0x08           ; DRQ bit
    jz .read_wait
    test al, 0x01           ; ERR bit
    jnz .ata_error

    ; Read 256 words (512 bytes) from the data port (0x1F0)
    mov cx, 256
    mov dx, 0x1F0
    rep insw                ; Read word from DX to [DS:SI]

    popa
    ret

section .bss
read_buffer resb 512        ; Buffer to hold the read sector
```

## So, What's The Catch? (Again)

1.  **Privileges**: Requires direct hardware access. Impossible from userland (Ring 3) on any modern OS (NT-based Windows, Linux, macOS). You need a kernel driver or a DOS environment.
2.  **Hardware Abstraction**: Modern hardware uses **AHCI (Advanced Host Controller Interface)** instead of the legacy IDE controller. The communication is completely different and involves memory-mapped I/O and **FIS (Frame Information Structures)**, not simple IN/OUT instructions.
3.  **Drive Security**: Modern SSDs and HDDs often have their own microcontrollers and security features (e.g., TCG Opal, Sanitize commands) that make low-level manipulation much harder and less universal.
4.  **Virtualization**: VMs abstract the hardware. Talking to "0x1F0" in a VM talks to a *virtual* controller, not the physical one.

## CTF & Practical Applications

This is pure **hardware hacking** and **forensics/anti-forensics** territory.
*   **Bootkit Development**: Malware that persists in the MBR or VBR (Volume Boot Record) uses these techniques. Famous examples: **TDL4**, **Rovnix**.
*   **Data Recovery**: Sometimes you need to send vendor-specific `SMART` commands or perform a low-level format.
*   **CTF Challenges**: Look for "hardware" or "forensics" challenges involving disk images, recovering data from a corrupted drive, or finding hidden data in the service area. The **Cyber Apocalypse CTF 2021** had hardware challenges involving SPI and UART, which are in the same low-level spirit .

## References & Further Reading

1.  **[ATA/ATAPI Command Set (ACS)](https://www.t13.org/documents/UploadedDocuments/docs2013/d2168r5-ATAATAPI_Command_Set_-_3.pdf)** - The official, mind-numbing specification. Chapter 7 is your friend.
2.  **[OSDev Wiki: ATA PIO Mode](https://wiki.osdev.org/ATA_PIO_Mode)** - A more practical guide for programmers.
3.  **"Rootkits: Subverting the Windows Kernel" by Hoglund & Butler** - Covers MBR/VBR rootkits.
4.  **"The PC Boot Process"** - Understanding how the MBR works is key.

