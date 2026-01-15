# Case 1: Port Scan Detection and Investigation

## Incident Summary

A port scanning activity was simulated from a Kali Linux system targeting a Windows 11 machine in a controlled lab environment. The objective was to analyze reconnaissance behavior and evaluate detection visibility.

---

## Lab Environment

* Hypervisor: VirtualBox
* Attacker: Kali Linux (192.168.1.18)
* Victim: Windows 11 (192.168.1.16)
* Tools: Nmap, Wireshark, Event Viewer, Sysmon
* Network Type: Host-only / Internal network

---

## Attack Simulation

The following command was executed on the Kali Linux system:

```
nmap -sS -T4 192.168.1.16
```

This command performs a stealth SYN scan to identify open ports on the target system.

---

## Detection Sources

| Source                | Result                             |
| --------------------- | ---------------------------------- |
| Wireshark             | SYN packets observed from attacker |
| Event Viewer          | No relevant entries detected       |
| Sysmon                | No network scan related logs       |
| Windows Firewall Logs | Not enabled                        |

---

## Evidence Collected

Screenshots included:
*Kali and Windows OS IP Addresses
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/25f32343-b224-4f43-83d5-3f2ea3465e25" />

* Kali Nmap scan output
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/dbed7067-3f8b-4709-9002-10240a865ffd" />


* Wireshark packet capture showing SYN packets
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/46af4a7a-0784-4bdf-b6fc-c1a6e6b8cfe4" />

* Event Viewer showing no corresponding logs
<img width="1282" height="1600" alt="Screenshot 2026-01-15 134615" src="https://github.com/user-attachments/assets/fce6a13b-2cde-4091-ac7d-b54a10ffaebf" />



(All screenshots are stored in the `screenshots` folder.)

---

## Analysis

The stealth port scan did not generate host-based logs because:

* No service interaction occurred.
* No full TCP handshake was completed.
* Sysmon primarily logs internal OS events.
* Windows did not register the scan as a connection.

This demonstrates that host-based logging alone is insufficient to detect stealth reconnaissance activity.

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name            |
| ------------ | ------------------------- |
| T1046        | Network Service Discovery |

---

## Impact Assessment

An attacker could perform reconnaissance without triggering host-based detection mechanisms, allowing them to map network services silently.

---

## Recommendations

* Enable firewall connection logging.
* Deploy network IDS/IPS.
* Integrate network telemetry into SIEM.
* Implement layered security monitoring.

---

## Conclusion

This case demonstrates the importance of combining network and endpoint monitoring. Stealth reconnaissance techniques can bypass endpoint logging, creating a detection gap if network visibility is not implemented.

---

## Analyst Notes

This investigation highlights real-world SOC challenges where absence of logs does not indicate absence of malicious activity.
