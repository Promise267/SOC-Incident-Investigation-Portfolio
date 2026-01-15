# Case 2 – SSH Brute Force Login Attempt Investigation (Windows OpenSSH)

## Incident Title

SSH Brute Force Authentication Attempt Against Windows OpenSSH Server

---

## Executive Summary

In this case study, a Windows 11 system was intentionally configured with an OpenSSH server and a local user account to simulate a realistic enterprise scenario. A Kali Linux attacker machine then attempted to gain unauthorized access using a brute force password attack via Hydra. Multiple failed SSH authentication attempts were detected in Windows Security logs, confirming brute force behavior. The attack was rate-limited and prevented before any successful compromise occurred.

---

## Lab Environment

* VirtualBox
* Kali Linux (Attacker)
* Windows 11 (Victim)
* OpenSSH Server (Windows Feature)
* Event Viewer (Security Logs)
* Hydra (Kali Linux)

---

## Environment Preparation

### Windows 11 (Victim)

1. Installed OpenSSH Server using PowerShell.
2. Started and enabled the SSH service (sshd).
3. Opened TCP port 22 in Windows Firewall.
4. Created a local user account:

   ```powershell
   net user socuser Password123 /add
   ```
5. Confirmed SSH service was listening on port 22.

<img width="2560" height="1600" alt="Screenshot 2026-01-14 141845" src="https://github.com/user-attachments/assets/43af312f-a35a-4526-9ec9-cba80ab94918" />


### Kali Linux (Attacker)

Kali Linux was connected to the same internal network as the Windows VM and used to perform the brute force attack using Hydra.

---

## Attack Scenario

The attacker identified that the Windows host exposed an SSH service on TCP port 22. Assuming the attacker had obtained a valid username through OSINT or prior exposure, a password brute force attack was launched against the `socuser` account using a known leaked password wordlist.


---

## Attack Simulation Command

```bash
hydra -l socuser -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.16 -t 4 -V
```

This command instructed Hydra to repeatedly attempt SSH authentication using common passwords against the Windows OpenSSH server.

<img width="1249" height="1502" alt="image" src="https://github.com/user-attachments/assets/b47df63b-7661-48bf-9d55-a35ee88395bd" />

---

## Detection Evidence

### Windows Security Log

**Log Source:** Event Viewer → Windows Logs → Security
**Event ID:** 4625
**Description:** An account failed to log on

<img width="2560" height="1600" alt="Screenshot 2026-01-15 165947" src="https://github.com/user-attachments/assets/956d2aa7-3e7f-4e41-b91a-afbaf7b544f5" />


Key Fields Observed:

* Target Username: socuser / FakeUser
* Process Name: sshd.exe
* Logon Type: 8
* Failure Reason: Unknown user name or bad password

These events confirm repeated failed SSH authentication attempts processed by the Windows OpenSSH service. The Windows Security log served as the primary detection source for this investigation.


## Analysis

The attacker attempted multiple password guesses against a known Windows SSH user account using a leaked password wordlist. Repeated Event ID 4625 entries in the Windows Security log confirmed brute force authentication behavior. The presence of multiple failed attempts within a short time window indicates automated password guessing rather than legitimate user error.

The SSH service limited incoming connections, preventing successful authentication. No account compromise or privilege escalation was observed.

---

## MITRE ATT&CK Mapping

| Technique         | ID        |
| ----------------- | --------- |
| Brute Force       | T1110     |
| Password Guessing | T1110.001 |
| Valid Accounts    | T1078     |

---

## Impact Assessment

* No successful authentication
* No data access
* No persistence
* System integrity maintained

---

## Incident Classification

**Severity:** Medium
**Category:** Credential Access Attempt
**Status:** Prevented

---

## Root Cause

Exposure of SSH service combined with password-based authentication created an opportunity for brute force exploitation.

---

## Recommendations

* Enforce strong password policies
* Enable account lockout thresholds
* Implement SSH key-based authentication
* Restrict SSH access to trusted IPs
* Configure SIEM alerts for repeated Event ID 4625 occurrences

---

## Conclusion

This case demonstrates a realistic SOC investigation workflow using Windows Security authentication logs to detect and analyze a brute force SSH attack against a Windows OpenSSH server. Proper monitoring of Event ID 4625 allowed accurate identification of credential attack behavior and confirmed that system protections successfully prevented unauthorized access.

