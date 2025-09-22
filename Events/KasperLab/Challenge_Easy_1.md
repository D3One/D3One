
## **Malware Reverse Engineering Task: Analysis of `Backdoor.Win32.Agent.gen`**

<img width="1664" height="928" alt="image" src="https://github.com/user-attachments/assets/dca757b2-60d5-4bc8-b647-5a248da13401" />

#### **Task Overview**
- **Sample:** `sample_malware.exe` (SHA-256: `a1b2c3d4e5f67890...`)  
- **Source:** Provided by Kaspersky Lab, 2017  
- **Time Limit:** 2 hours  
- **Difficulty:** Easy  
- **Objective:** Analyze the binary to determine its functionality, identify key assembly patterns, Windows API calls, and propose mitigation strategies.  

---

### **Technical Analysis Steps**

#### **1. Assembly Code Analysis**
**Code Snippet: File Creation and Persistence**  
```asm
; Set up stack frame
PUSH EBP
MOV EBP, ESP
SUB ESP, 60h        ; Allocate space for local variables

; Call CreateFileA to write malicious executable
PUSH 0               ; hTemplateFile
PUSH 80h             ; FILE_ATTRIBUTE_NORMAL
PUSH 2               ; CREATE_ALWAYS
PUSH 0               ; lpSecurityDescriptor
PUSH 0               ; dwShareMode
PUSH 0C0000000h      ; GENERIC_READ | GENERIC_WRITE
PUSH offset FileName ; Pointer to "%AppData%\system.exe"
CALL CreateFileA
MOV [ebp+hFile], EAX ; Save file handle

; Check for errors
CMP EAX, INVALID_HANDLE_VALUE
JZ error_handler     ; Jump if ZF=1 (operation failed)
```
**Key Instructions:**  
- `MOV EBP, ESP`: Sets up the stack frame for local variable access.  
- `PUSH`: Pushes arguments onto the stack for API calls.  
- `CALL`: Invokes Windows API functions (e.g., `CreateFileA`).  
- `CMP`/`JZ`: Compares values and jumps based on the Zero Flag (`ZF`).  

#### **2. Registry Modification for Persistence**  
```asm
; Open registry key for autostart
PUSH offset KeyPath   ; "Software\Microsoft\Windows\CurrentVersion\Run"
PUSH 0F003Fh          ; KEY_SET_VALUE | KEY_WOW64_64KEY
CALL RegOpenKeyExA
TEST EAX, EAX         ; Check result
JNZ exit_loop         ; Jump if not zero (error)

; Set registry value
PUSH 12h              ; Data size
PUSH offset FilePath  ; Malware path
PUSH 1                ; REG_SZ
PUSH 0                ; Reserved
PUSH offset ValueName ; "SystemLoader"
PUSH [ebp+hKey]       ; Registry handle
CALL RegSetValueExA
```
**Flags and Branches:**  
- `TEST EAX, EAX`: Sets `ZF=1` if `EAX=0` (success).  
- `JNZ`: Jumps if `ZF=0`, indicating an error.  

#### **3. Network Communication with C&C**  
```asm
; Connect to C&C server
PUSH 0               ; dwFlags
PUSH 0               ; dwReserved
PUSH 443             ; Port (HTTPS)
PUSH offset ServerIP ; "185.243.115.230"
CALL WinHttpConnect
MOV [ebp+hSession], EAX

; Check response status
CALL WinHttpReceiveResponse
TEST EAX, EAX
JZ connection_failed ; Jump if ZF=1 (failure)

CMP [dwStatusCode], 200
JNZ invalid_response ; Jump if not 200
```
**Flag Usage:**  
- `TEST EAX, EAX`: Sets `ZF` based on whether `EAX=0`.  
- `JZ`/`JNZ`: Control flow based on `ZF`.  

---

### **Key Findings**
1. **Functionality:**  
   - Creates a copy in `%AppData%\system.exe`.  
   - Adds persistence via registry key `HKLM\...\Run`.  
   - Communicates with C&C server `185.243.115.230:443`.  

2. **API Calls:**  
   - `CreateFileA`: Writes files.  
   - `RegSetValueExA`: Modifies registry.  
   - `WinHttpConnect`: Establishes network connections.  

3. **Assembly Patterns:**  
   - Stack manipulation using `PUSH`/`POP`.  
   - Conditional jumps (`JZ`, `JNZ`) based on `ZF`.  
   - Data transfer via `MOV` and `LEA`.  

---

### **Mitigation Strategies**
1. **Behavioral Detection:**  
   - Monitor writes to `%AppData%` and `HKLM\...\Run`.  
   - Block network traffic to suspicious IPs (e.g., `185.243.115.230`).  

2. **Static Indicators:**  
   - File hash: `a1b2c3d4e5f67890...`.  
   - Strings: `"SystemLoader"`, `"185.243.115.230"`.  

3. **Tools:**  
   - Use **IDA Pro** for disassembly.  
   - Monitor with **Wireshark** for C&C traffic.  

---

### **Practical Exercises**
1. Trace execution from entry point to `CreateFileA`.  
2. Identify conditions triggering `error_handler`.  
3. Analyze how `ZF` is used after `WinHttpReceiveResponse`.  

**Deliverable:** Submit a report detailing functionality, API calls, and mitigation measures.
