### **Sample 1: Simple CrackMe (Beginner Level)**

**Technical Overview:** This is a straightforward console application compiled from C without optimizations. The password is hardcoded and compared using the standard `strcmp` library function. The key is to locate this comparison and either extract the password or patch the conditional jump.

**Annotated Disassembly (IDA Pro View):**

```asm
; SECTION .text: CODE (Address: 0x00401000)
; --- Main Function ---
start:
    push    ebp
    mov     ebp, esp
    sub     esp, 58h                ; Allocate space for local variables
    and     esp, 0FFFFFFF0h         ; Align stack to 16 bytes
    mov     eax, 0
    call    __alloca_probe          ; MSVC stack allocation helper

    push    offset aEnterPassword ; "Enter password: "
    call    printf
    add     esp, 4

    lea     eax, [ebp+user_input]   ; EAX = address of user_input buffer
    push    eax
    push    offset aS         ; Format string "%s"
    call    scanf
    add     esp, 8

    lea     eax, [ebp+user_input]   ; Push user input
    push    eax
    push    offset hardcoded_pass ; Push hardcoded password "s3cr3t_p@55"
    call    _strcmp
    add     esp, 8

    test    eax, eax                ; Check strcmp result (0 = equal)
    jnz     short bad_password      ; Jump if Not Zero (passwords don't match)

good_password:
    push    offset aAccessGranted ; "Access Granted! Flag: {%s}\n"
    push    offset hardcoded_pass ; Also use the password as the flag
    call    printf
    add     esp, 8
    jmp     short exit_program

bad_password:
    push    offset aAccessDenied  ; "Access Denied!\n"
    call    printf
    add     esp, 4

exit_program:
    mov     eax, 0                  ; Return 0
    mov     esp, ebp
    pop     ebp
    retn

; SECTION .data: DATA (Address: 0x00403000)
hardcoded_pass db 's3cr3t_p@55',0   ; The password we need to find
aEnterPassword db 'Enter password: ',0
aAccessGranted db 'Access Granted! Flag: {%s}',0Ah,0
aAccessDenied  db 'Access Denied!',0Ah,0
aS          db '%s',0
```

**Key Instructions in OllyDbg:**
1.  The `CALL <&MSVCRT.strcmp>` at address `0x00401045` is the critical comparison.
2.  Set a breakpoint here. When hit, look at the stack. The two arguments will be visible:
    *   `0012FF5C` -> Your input (e.g., "test")
    *   `00403000` -> The hardcoded string `"s3cr3t_p@55"`
3.  The `JNZ SHORT 0040105A` instruction right after the `test eax, eax` is the decision point. Changing this `JNZ` (Jump if Not Zero) to a `NOP` (90) or a `JZ` (Jump if Zero) will alter the program's logic.

---

### **Sample 2: Modified UPX Packed Binary (Intermediate Level)**

**Technical Overview:** The binary is packed with a modified UPX. The entry point leads to the unpacking routine. The goal is to let the unpacker run, find the Original Entry Point (OEP), dump the process from memory, and then analyze the real code.

**Annotated Unpacking Stub (Start of the binary in OllyDbg):**

```asm
; --- UPX Unpacking Stub (Modified) ---
; This code is responsible for decompressing the original .text section into memory.
start:
    pushad                          ; Save all registers (marks start of unpacking)
    mov     esi, 0040E000h          ; Source: compressed data
    mov     edi, 00401000h          ; Destination: where original .text will be unpacked
    mov     ecx, 0000C000h          ; Size of the block to unpack
    mov     ebx, 12345678h          ; Key for modified UPX decryption (varies)
    cld                             ; Clear direction flag (move forward)

decompress_loop:
    lodsb                           ; Load a byte from [ESI] into AL
    xor     al, bl                  ; Decrypt the byte with a simple XOR
    stosb                           ; Store AL into [EDI]
    loop    decompress_loop         ; Loop until ECX is 0

    ; The following is a common UPX trick to hide the jump to OEP
    popad                           ; Restore all registers
    jmp     near ptr original_entry_point ; **This is the jump to the OEP**

; After the JMP, the original unpacked code will be at 0x00401000
```

**Finding the OEP:**
1.  The unpacking stub ends with a `POPAD` and then a far `JMP`.
2.  In this case, the `JMP` goes to `0x004012D0`. This is the **Original Entry Point (OEP)**.
3.  Let the program run until this `JMP` is executed (use F7 to trace into it). You are now in the "real" code of the program.

**Annotated Code of the Unpacked Binary (After Dumping):**

```asm
; --- Unpacked Code at OEP (0x004012D0) ---
original_entry_point:
    push    ebp
    mov     ebp, esp
    sub     esp, 10h
    push    offset aWelcomeToCrackm ; "Welcome to CrackMe v2.0"
    call    printf
    add     esp, 4
    ; ... (code similar to Sample 1, but with a twist) ...

check_password:
    mov     esi, offset user_input
    mov     edi, offset real_password ; real_password = "p@ck3d_c0rrect"
    mov     ecx, 0Eh                 ; Length of the password to compare

compare_loop:                        ; Uses a custom loop, not strcmp!
    lodsb                           ; Load byte from [ESI] into AL, inc ESI
    scasb                           ; Compare AL with byte at [EDI], inc EDI
    jnz     short bad_pass          ; Mismatch found
    loop    compare_loop            ; Loop until ECX is 0
    jmp     short good_pass

bad_pass:
    ; ... print error ...
good_pass:
    ; ... print success ...
    ; The flag might be calculated: "Flag_{" + real_password + "}"

; SECTION .data of the unpacked binary
real_password db 'p@ck3d_c0rrect',0 ; Found in the dumped binary's data section
```

---

### **Sample 3: Advanced KeygenMe (Hard Level)**

**Technical Overview:** This Win32 GUI application uses a custom algorithm to validate a Name/Serial combination. It includes anti-debugging techniques and requires full algorithm reversal to write a keygen.

**Annotated Validation Function (IDA Pro Pseudocode + ASM):**

```c
// IDA Pro's decompiled C view of the validation function (simplified)
BOOL __stdcall ValidateSerial(char *name, char *serial) {
    int name_len;
    int checksum;
    int calculated_serial;
    int i;

    // Simple anti-debugging trick
    if (IsDebuggerPresent()) {
        MessageBoxA(0, "Debugger detected!", "Error", MB_ICONERROR);
        return FALSE;
    }

    name_len = lstrlenA(name);
    if (name_len < 4) {
        return FALSE; // Name must be at least 4 chars
    }

    checksum = 0;
    // Calculate a checksum from the name
    for (i = 0; i < name_len; i++) {
        checksum = (checksum * 0x21) + name[i]; // Basic rolling hash
        checksum = checksum & 0xFFFFFFFF; // Keep it 32-bit
    }

    // The core algorithm: complex transformation
    calculated_serial = (checksum ^ 0xCAFEBABE) * 0x1337;
    calculated_serial = calculated_serial & 0xFFFFFFFF;

    // Compare the calculated integer to the user's input string
    return (atoi(serial) == calculated_serial); // User input is converted to an int
}
```

**Corresponding Assembly Snippets of Critical Parts:**

```asm
; Anti-Debugging Check
.text:004011D0    call    ds:IsDebuggerPresent
.text:004011D6    test    eax, eax
.text:004011D8    jz      short no_debugger
.text:004011DA    push    0               ; uType = MB_OK
.text:004011DC    push    offset Title    ; "Error"
.text:004011E1    push    offset Text     ; "Debugger detected!"
.text:004011E6    push    0               ; hWnd = NULL
.text:004011E8    call    ds:MessageBoxA
.text:004011EE    xor     eax, eax        ; Return FALSE
.text:004011F0    jmp     short loc_40126A

; The Core Algorithm Loop
.text:00401210    mov     ecx, [ebp+name_len]
.text:00401213    mov     esi, [ebp+name]
.text:00401216    xor     eax, eax        ; i = 0
.text:00401218    xor     ebx, ebx        ; checksum = 0
.text:0040121A
.text:0040121A calc_loop:
.text:0040121A    movsx   edx, byte ptr [esi+eax] ; Load next char from name
.text:0040121E    imul    ebx, 21h        ; checksum = checksum * 33
.text:00401221    add     ebx, edx        ; checksum = checksum + char
.text:00401223    inc     eax             ; i++
.text:00401224    cmp     eax, ecx
.text:00401226    jl      short calc_loop

; Final Transformation
.text:00401238    mov     eax, ebx        ; eax = checksum
.text:0040123A    xor     eax, 0CAFEBABEh ; XOR with magic constant
.text:0040123F    imul    eax, 1337h      ; Multiply by magic constant
.text:00401245    mov     [ebp+calculated_serial], eax
```

**Keygen Code (Python):**

```python
def generate_key(name):
    if len(name) < 4:
        return "Name too short!"
    checksum = 0
    for c in name:
        checksum = (checksum * 0x21) + ord(c)
        checksum &= 0xFFFFFFFF  # Simulate 32-bit integer
    serial = (checksum ^ 0xCAFEBABE) * 0x1337
    serial &= 0xFFFFFFFF
    return str(serial)

name = input("Enter your name: ")
print("Your serial is: " + generate_key(name))
```
