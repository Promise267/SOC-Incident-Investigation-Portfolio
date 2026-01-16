# Metasploit Reverse Shell Lab – Session Drop Analysis

## Environment Overview

**Figure 1 – Lab Network Topology**
*Description:* Kali Linux attacker (192.168.1.18) and Windows 11 victim (192.168.1.11) on the same subnet.

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/06792b4f-768a-4d02-a6b0-dc68533ef43b" />


---

**Attacker Machine (Kali Linux)**
IP Address: 192.168.1.18
Tool: Metasploit Framework (msfconsole)
Listener Type: Windows Meterpreter reverse TCP handler
Listening Port: 4444

**Victim Machine (Windows 11)**
IP Address: 192.168.1.11
Protection: Windows Defender and Firewall disabled (for lab testing)

---

## Test Scenario

**Figure 2 – Payload Hosting on Kali**
*Description:* Web server hosting payload.exe on 192.168.1.18.

<img width="2560" height="1600" alt="Screenshot (29)" src="https://github.com/user-attachments/assets/e2dcdac2-3083-466e-bb17-6e47eccde3b1" />


---

A reverse shell listener was started on the Kali Linux system using Metasploit’s multi/handler module. The victim Windows 11 system accessed the payload through a browser by navigating to:

```
<img width="2560" height="1600" alt="Screenshot (28)" src="https://github.com/user-attachments/assets/1e8edd5a-1402-469b-a6c8-0af8a99b091c" />

```

After execution of the payload on Windows 11, a Meterpreter session was successfully established on Kali Linux. However, the session terminated shortly after connection.

---

## Observed Behavior

**Figure 3 – Initial Meterpreter Session**
*Description:* Meterpreter session successfully opened in Metasploit.

<img width="2560" height="1600" alt="Screenshot (29)" src="https://github.com/user-attachments/assets/e2dcdac2-3083-466e-bb17-6e47eccde3b1" />

---

* Initial connection from Windows 11 to Kali was successful.
* A Meterpreter session opened in Metasploit.
* The session closed automatically after a short time.
* No persistence mechanism was configured.

---

## Likely Reasons for Session Termination

**Figure 4 – Session Termination Event**
*Description:* Meterpreter session closing unexpectedly.

<img width="2560" height="1600" alt="Screenshot (29)" src="https://github.com/user-attachments/assets/e2dcdac2-3083-466e-bb17-6e47eccde3b1" />

---

### 1. Windows Defender Residual Protection

Even when Defender appears disabled, Windows 11 may still terminate suspicious processes using:

* AMSI
* SmartScreen
* Memory protection
* Cloud-based reputation checks

### 2. Payload Stability

Some payloads are unstable on modern Windows 11 builds due to:

* Process injection failures
* Privilege mismatch
* Compatibility issues

### 3. Network Timeout or Listener Mismatch

If the payload reconnect interval or handler configuration is incorrect, the session can drop.

### 4. Process Termination

The process hosting the payload may be automatically terminated by Windows after execution.

---

## Professional Interpretation

This behavior accurately reflects real-world post-exploitation challenges. Establishing an initial session does not guarantee persistence or stability. Modern operating systems aggressively defend against reverse shells even when visible protections are disabled.

---

## Defensive Learning Outcome

**Figure 5 – Windows Security Configuration**
*Description:* Windows Defender and Firewall disabled for testing.

<img width="2560" height="1600" alt="Screenshot (22)" src="https://github.com/user-attachments/assets/0557d646-0853-4b41-90cf-07343c01b249" />


---

From a cybersecurity perspective, this demonstrates:

* Effectiveness of modern endpoint protections
* Importance of persistence techniques
* Challenges in maintaining command-and-control sessions
* Why attackers rely on stealth, not just connection success

---

## Project Conclusion Statement

> The lab demonstrated that while a reverse Meterpreter session could be successfully established in a controlled environment, session stability remained limited due to modern operating system protection mechanisms. This highlights the importance of endpoint detection and response technologies in mitigating post-exploitation persistence.

---

## Future Work

* Investigate session persistence mechanisms
* Test different payload formats
* Observe Defender logs and AMSI behavior
* Compare session stability across Windows versions

---

## Ethical Note

All testing was conducted in a controlled virtual lab environment on systems owned by the tester for educational cybersecurity research purposes.
