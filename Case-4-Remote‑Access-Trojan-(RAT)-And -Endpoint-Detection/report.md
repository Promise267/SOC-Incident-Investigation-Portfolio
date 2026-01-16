# Metasploit Reverse Shell Lab – Session Drop Analysis

## Environment Overview

**Figure 1 – Lab Network Topology**  
*Description:* Kali Linux attacker (192.168.1.18) and Windows 11 victim (192.168.1.11) on the same subnet.

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/06792b4f-768a-4d02-a6b0-dc68533ef43b" />

---

**Attacker Machine (Kali Linux)**  
- IP Address: `192.168.1.18`  
- Tool: Metasploit Framework (`msfconsole`)  
- Listener Type: Windows Meterpreter reverse TCP handler  
- Listening Port: `4444` 

**Victim Machine (Windows 11)**  
- IP Address: `192.168.1.11`  
- Protection (phase 1): Windows Defender and Firewall disabled for lab testing, then re‑enabled to observe detection. 
---

## Test Scenario

**Figure 2 – Payload Hosting on Kali**  
*Description:* Web server hosting `payload.exe` on 192.168.1.18.

<img width="2560" height="1600" alt="Screenshot (29)" src="https://github.com/user-attachments/assets/e2dcdac2-3083-466e-bb17-6e47eccde3b1" />

A reverse shell listener was started on the Kali Linux system using Metasploit’s `exploit/multi/handler` module. The victim Windows 11 system accessed the payload through a browser by navigating to the Kali web server and then executed `payload.exe` from `C:\Users\vboxuser\Downloads\`. 

<img width="2560" height="1600" alt="Screenshot (28)" src="https://github.com/user-attachments/assets/8c14f266-45d4-4e5c-816b-d44d1fb1846a" />

After execution of the payload on Windows 11, a Meterpreter session was successfully established on Kali Linux. However, the session terminated shortly after connection.

**Relevant MITRE ATT&CK techniques (simulated)**  
- **T1059 – Command and Scripting Interpreter:** use of Metasploit console and command‑line tooling. 
- **T1204.002 – User Execution: Malicious File:** user runs `payload.exe` on Windows. 
- **T1105 – Ingress Tool Transfer:** payload is delivered from Kali to the Windows host over HTTP.

---

## Observed Behavior

**Figure 3 – Initial Meterpreter Session**  
*Description:* Meterpreter session successfully opened in Metasploit, then closed with “Reason: Died”.

<img width="2560" height="1600" alt="Screenshot (29)" src="https://github.com/user-attachments/assets/e2dcdac2-3083-466e-bb17-6e47eccde3b1" />

- Initial connection from Windows 11 to Kali was successful.  
- A Meterpreter session opened in Metasploit.  
- The session closed automatically after a short time with `Reason: Died`.  
- No persistence mechanism was configured, so losing the process meant losing access.

**Relevant MITRE ATT&CK techniques**  
- **T1055 – Process Injection (conceptually):** Meterpreter uses in‑memory techniques to run on the host, similar to process injection used by many RATs. 
- **T1105 – Ingress Tool Transfer:** the Meterpreter stage is sent from Kali to the victim as part of the connection. 

---

## Windows Defender Log Evidence

When Microsoft Defender Antivirus was enabled and the payload was executed again, Windows logged **malware detection and remediation events** under:

> `Applications and Services Logs → Microsoft → Windows → Windows Defender → Operational`

This is an example of the **Detection** side of the MITRE ATT&CK framework, where endpoint security tools generate telemetry for malicious activity.

### Event ID 1116 – Malware detected

**Figure 4 – Event 1116 (Detection)**  
*Description:* Defender detects `payload.exe` as `Trojan:Win32/Meterpreter.RPZ!MTB` while executing.

<img width="2560" height="1600" alt="Screenshot (34)" src="https://github.com/user-attachments/assets/90da88de-56c1-4e70-8f7e-699c0d0c2256" />


Key fields:

- Threat Name: `Trojan:Win32/Meterpreter.RPZ!MTB`  
- Category: Trojan  
- Severity: Severe  
- Process Name: `C:\Users\vboxuser\Downloads\payload.exe`

This shows Defender detecting the Meterpreter reverse‑shell executable as a Trojan / remote access trojan (RAT). 

**Mapped ATT&CK techniques (adversary behaviour)**  
- **T1219 – Remote Access Software:** Meterpreter provides full remote access to the victim system. 
- **T1059 – Command and Scripting Interpreter:** commands are issued through the Meterpreter session.

**Mapped ATT&CK data sources / detections**  
- **Process Monitoring / Anti‑virus logs:** Defender’s Event ID 1116 records the malicious process and detection result.

---

### Event ID 1117 – Action taken (quarantine)

**Figure 5 – Event 1117 (Quarantine)**  
*Description:* Defender quarantines `payload.exe`, terminating the Meterpreter session.

<img width="2560" height="1600" alt="Screenshot (35)" src="https://github.com/user-attachments/assets/23b96256-8aa9-45f5-bed4-11a661ff4963" />


Key fields:

- Action Name: `Quarantine`  
- State: 2 (remediation)  
- Error Code: `0x00000000` – remediation completed successfully

This confirms Defender quarantined the payload and killed the process, which explains why the Meterpreter session dropped shortly after opening.

**Mapped ATT&CK mitigations / detections**  
- **M1049 – Antivirus / Antimalware:** Defender detects and removes the RAT. 
- **M1037 – Filter Network Traffic:** the reverse connection is effectively interrupted once the process is terminated.

---

## Likely Reason for Session Termination

The Event ID 1116 and 1117 entries indicate that **Microsoft Defender Antivirus**, not network issues, was responsible for the session drop:

- Defender detected the Meterpreter executable as `Trojan:Win32/Meterpreter.RPZ!MTB`. 
- Defender then quarantined the executable, terminating the process that hosted the Meterpreter session. 

Other possible factors (payload stability, privilege mismatch, network timeouts) can affect shells in general, but in this lab the logs clearly show Defender as the primary cause. 

From a MITRE ATT&CK perspective, this is a defender successfully disrupting technique **T1219 – Remote Access Software** by using host‑based security controls.

---

## Professional Interpretation

This behavior accurately reflects real‑world post‑exploitation challenges. Establishing an initial session (**TA0008 – Lateral Movement / TA0010 – Exfiltration infrastructure setup**) does not guarantee persistence or stability; modern endpoint protections are designed to detect and remove remote‑access tools such as Meterpreter quickly after execution. 

---

## Defensive Learning Outcome

**Figure 6 – Windows Security Configuration**  
*Description:* Windows Defender and Firewall disabled for the initial test, then re‑enabled for detection.

<img width="2560" height="1600" alt="Screenshot (22)" src="https://github.com/user-attachments/assets/0557d646-0853-4b41-90cf-07343c01b249" />

From a cybersecurity perspective, this demonstrates:

- How Microsoft Defender logs Event IDs **1116** (malware detected) and **1117** (action taken) for Meterpreter‑style payloads, giving visibility into **T1219 – Remote Access Software** and **T1204 – User Execution**.  
- The effectiveness of modern endpoint protections (**M1049 – Antivirus/Antimalware**) in killing active reverse shells once real‑time protection is enabled.
- The importance of persistence techniques (**T1547 – Boot or Logon Autostart Execution**) and stealth for attackers who want to maintain command‑and‑control sessions after the initial compromise.   
- Why defenders should monitor these specific Defender events as part of endpoint detection and response (ATT&CK data sources: **Process Monitoring**, **Application Log**).

---

## Project Conclusion Statement

> The lab demonstrated that while a reverse Meterpreter session (mapping to MITRE ATT&CK **T1219 – Remote Access Software**) could be successfully established in a controlled environment, Microsoft Defender Antivirus quickly detected the payload as `Trojan:Win32/Meterpreter.RPZ!MTB` (Event 1116) and quarantined it (Event 1117), causing the session to terminate. This highlights the importance of endpoint detection and response technologies and ATT&CK‑aligned controls such as **M1049 – Antivirus/Antimalware** in mitigating post‑exploitation persistence on modern Windows systems. 

---

## Future Work

- Investigate session persistence mechanisms (e.g., registry run keys, services) that map to **T1547 – Boot or Logon Autostart Execution**, and observe how they appear in Defender and Windows logs. 
- Test different payload formats (e.g., `reverse_https`, stageless vs. staged) and compare detection results for techniques such as **T1105 – Ingress Tool Transfer** and **T1219 – Remote Access Software**. 
- Observe additional telemetry such as AMSI and PowerShell logs, relevant to **T1059 – Command and Scripting Interpreter** and **T1562 – Impair Defenses** when attackers try to tamper with security tools. 
- Compare session stability and detection outcomes across different Windows versions and configurations to understand how well various platforms resist ATT&CK techniques like **T1204**, **T1219**, and **T1055**.

---
