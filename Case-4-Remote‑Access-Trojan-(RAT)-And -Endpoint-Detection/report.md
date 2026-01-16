# Metasploit Reverse Shell Lab – Session Drop Analysis

## Environment Overview

**Figure 1 – Lab Network Topology**  
*Description:* Kali Linux attacker (192.168.1.18) and Windows 11 victim (192.168.1.11) on the same subnet. 

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/06792b4f-768a-4d02-a6b0-dc68533ef43b" />

---

**Attacker Machine (Kali Linux)**  
- IP Address: 192.168.1.18  
- Tool: Metasploit Framework (`msfconsole`)  
- Listener Type: Windows Meterpreter reverse TCP handler  
- Listening Port: 4444 

**Victim Machine (Windows 11)**  
- IP Address: 192.168.1.11  
- Protection (phase 1): Windows Defender and Firewall disabled for lab testing, then re‑enabled to observe detection. 

---

## Test Scenario

**Figure 2 – Payload Hosting on Kali**  
*Description:* Web server hosting `payload.exe` on 192.168.1.18.

<img width="2560" height="1600" alt="Screenshot (29)" src="https://github.com/user-attachments/assets/e2dcdac2-3083-466e-bb17-6e47eccde3b1" />

A reverse shell listener was started on the Kali Linux system using Metasploit’s `exploit/multi/handler` module. The victim Windows 11 system accessed the payload through a browser by navigating to the Kali web server and then executed `payload.exe` from `C:\Users\vboxuser\Downloads\`. 

<img width="2560" height="1600" alt="Screenshot (28)" src="https://github.com/user-attachments/assets/8c14f266-45d4-4e5c-816b-d44d1fb1846a" />

After execution of the payload on Windows 11, a Meterpreter session was successfully established on Kali Linux. However, the session terminated shortly after connection.

---

## Observed Behavior

**Figure 3 – Initial Meterpreter Session**  
*Description:* Meterpreter session successfully opened in Metasploit, then closed with “Reason: Died”.

<img width="2560" height="1600" alt="Screenshot (29)" src="https://github.com/user-attachments/assets/e2dcdac2-3083-466e-bb17-6e47eccde3b1" />

- Initial connection from Windows 11 to Kali was successful.  
- A Meterpreter session opened in Metasploit.  
- The session closed automatically after a short time with `Reason: Died`.  
- No persistence mechanism was configured, so losing the process meant losing access.

---

## Windows Defender Log Evidence

When Microsoft Defender Antivirus was enabled and the payload was executed again, Windows logged **malware detection and remediation events** under:

> `Applications and Services Logs → Microsoft → Windows → Windows Defender → Operational` 

### Event ID 1116 – Malware detected

**Figure 4 – Event 1116 (Detection)**  
*Description:* Defender detects `payload.exe` as `Trojan:Win32/Meterpreter.RPZ!MTB` while executing.

<img width="2560" height="1600" alt="Screenshot (34)" src="https://github.com/user-attachments/assets/bead0dd4-01b1-4055-9466-5704427eb594" />


Key fields:

- Threat Name: `Trojan:Win32/Meterpreter.RPZ!MTB`  
- Category: Trojan  
- Severity: Severe  
- Process Name: `C:\Users\vboxuser\Downloads\payload.exe`

This shows Defender detecting the Meterpreter reverse‑shell executable as a Trojan / remote access trojan (RAT).

### Event ID 1117 – Action taken (quarantine)

<img width="2560" height="1600" alt="Screenshot (35)" src="https://github.com/user-attachments/assets/5ffb9215-8f40-4b71-a73e-cd96c1254012" />

*Description:* Defender quarantines `payload.exe`, terminating the Meterpreter session.

> _Insert your Event Viewer screenshot for 1117 here._

Key fields:

- Action Name: `Quarantine`  
- State: 2 (remediation)  
- Error Code: `0x00000000` – remediation completed successfully

This confirms Defender quarantined the payload and killed the process, which explains why the Meterpreter session dropped shortly after opening. 

---

## Likely Reason for Session Termination

The Event ID 1116 and 1117 entries indicate that **Microsoft Defender Antivirus**, not network issues, was responsible for the session drop:

- Defender detected the Meterpreter executable as `Trojan:Win32/Meterpreter.RPZ!MTB`. 
- Defender then quarantined the executable, terminating the process that hosted the Meterpreter session.

Other possible factors (payload stability, privilege mismatch, network timeouts) can affect shells in general, but in this lab the logs clearly show Defender as the primary cause. 

---

## Professional Interpretation

This behavior accurately reflects real‑world post‑exploitation challenges. Establishing an initial session does not guarantee persistence or stability; modern endpoint protections are designed to detect and remove remote‑access trojans such as Meterpreter quickly after execution. 

---

## Defensive Learning Outcome

**Figure 6 – Windows Security Configuration**  
*Description:* Windows Defender and Firewall disabled for the initial test, then re‑enabled for detection.

<img width="2560" height="1600" alt="Screenshot (22)" src="https://github.com/user-attachments/assets/0557d646-0853-4b41-90cf-07343c01b249" />

From a cybersecurity perspective, this demonstrates:

- How Microsoft Defender logs Event IDs **1116** (malware detected) and **1117** (action taken) for Meterpreter‑style payloads.
- The effectiveness of modern endpoint protections in killing active reverse shells once real‑time protection is enabled. 
- The importance of persistence techniques and stealth for attackers who want to maintain command‑and‑control sessions.   
- Why defenders should monitor these specific Defender events as part of endpoint detection and response. 

---

## Project Conclusion Statement

> The lab demonstrated that while a reverse Meterpreter session could be successfully established in a controlled environment, Microsoft Defender Antivirus quickly detected the payload as `Trojan:Win32/Meterpreter.RPZ!MTB` (Event 1116) and quarantined it (Event 1117), causing the session to terminate. This highlights the importance of endpoint detection and response technologies in mitigating post‑exploitation persistence on modern Windows systems.

---

## Future Work

- Investigate session persistence mechanisms (services, registry run keys, scheduled tasks) and observe how they appear in Defender and Windows logs.
- Test different payload formats (e.g., `reverse_https`, stageless vs. staged) and compare detection results. 
- Observe additional telemetry such as AMSI and PowerShell logs alongside Defender detections.   
- Compare session stability and detection outcomes across different Windows versions and security configurations.

---
