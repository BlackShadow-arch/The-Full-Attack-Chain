# ⚔️ The Full Attack Chain — Capstone Red Team Engagement

**Intern:** Ali Ahsan | **Roll No:** CSI-B1-427
**Program:** Cyberstar Cybersecurity Red Teaming Internship
**Instructor:** Umar Niaz
**Date:** 22 May 2026
**Type:** 12-Week Capstone Project — End-to-End Red Team Engagement

---

## 📌 Overview

This capstone project documents a **complete, end-to-end Red Team engagement** conducted inside a controlled lab environment. Starting from external reconnaissance and working all the way through to full **Domain Administrator compromise**, every phase of the modern cyber kill chain was executed, documented, and mapped to the **MITRE ATT&CK Framework**.

> The most critical finding: **at no point did any zero-day or advanced exploit drive the attack.** Every stage of compromise was achieved through misconfigurations, outdated software, and poor security hygiene — issues entirely preventable with standard security practices.

---

## 🗺️ Attack Chain Overview

```
Reconnaissance → Initial Access → Privilege Escalation → Persistence
       → Lateral Movement → AD Enumeration → Domain Dominance → Cleanup
```

| Phase | Technique | Outcome |
|-------|-----------|---------|
| Reconnaissance | Nmap (TCP/SYN/UDP), NSE Scripts, SMB Enum | 23 open ports mapped, 4 CRITICAL services found |
| Initial Access | vsftpd 2.3.4 Backdoor (CVE-2011-2523) | Instant root shell via Metasploit |
| Privilege Escalation | SUID nmap binary escape, EternalBlue (MS17-010) | Root on Linux, SYSTEM on Windows 7 |
| Persistence | SSH Key Backdoor, Scheduled Task | Survives reboots and password resets |
| Lateral Movement | SSH Tunneling, Pass-the-Hash, WMIExec | Reached isolated Windows 7 machine |
| Credential Dumping | Kiwi (in-memory Mimikatz), /etc/shadow | All NTLM hashes extracted, 100% disk-free |
| AD Enumeration | PowerView, BloodHound, SharpHound | Full domain object map + attack paths |
| Kerberos Attacks | AS-REP Roasting, Kerberoasting | Plaintext credentials recovered |
| Domain Dominance | LLMNR Poisoning, DCSync, Golden Ticket | Permanent domain-wide access forged |
| Cleanup | SSH key removal, schtasks delete, ACL revert | All systems returned to pre-test state |

---

## 📋 Task Breakdown

### Task 01 — Scoping & Planning Phase

A professional Red Team engagement begins with a defined **Rules of Engagement (RoE)** document.

**Objectives:**
- Identify all open ports and running services on the target
- Gain initial shell access via a remote exploit (vsftpd / Samba)
- Escalate privileges to root on the Linux target
- Establish persistence (SSH backdoor, scheduled tasks)
- Pivot into the internal network and reach the Windows 7 machine
- Dump credentials from the Windows machine
- Enumerate the Active Directory environment and identify attack paths
- Achieve Domain Admin status via Golden Ticket / DCSync
- Establish resilient forest-level persistence

**Success Criteria:**

| Objective | Success Metric | Week Completed |
|-----------|---------------|----------------|
| Network Reconnaissance | Full port/service map with vulnerability matrix | Week 02 |
| Initial Access | Remote shell obtained via Metasploit exploit | Week 03 |
| Privilege Escalation | Root/SYSTEM access confirmed | Week 05 |
| Credential Harvesting | NTLM hashes extracted and cracked | Week 04 |
| Lateral Movement | RDP / WMI access to Windows 7 machine | Week 04 |
| AD Enumeration | BloodHound attack paths identified | Week 07 |
| Domain Dominance | Domain Admin via Golden Ticket | Week 08 |
| Persistence | Multiple mechanisms survive password resets | Week 08 |

**Ethical Boundaries:**
- All testing conducted in a controlled, isolated lab environment
- No unauthorized access to any production system or third-party network
- All activities supervised by instructor Mr. Umar Niaz

---

### Task 02 — Execution of the Cyber Kill Chain

#### 🔍 Reconnaissance & Network Scanning

**Discovered Open Ports:**

| Port | Service | Version | Risk |
|------|---------|---------|------|
| 21/tcp | FTP | vsftpd 2.3.4 | CRITICAL — Backdoor |
| 22/tcp | SSH | OpenSSH 4.7p1 | Medium |
| 80/tcp | HTTP | Apache 2.2.8 | High |
| 139/445/tcp | SMB | Samba 3.0.20 | CRITICAL — RCE |
| 3306/tcp | MySQL | 5.0.51a | Medium |
| 5432/tcp | PostgreSQL | 8.3.x | Medium |
| 6667/tcp | IRC | UnrealIRCd | CRITICAL — Backdoor |
| 8009/tcp | AJP | Apache JServ | High — Ghostcat |

**CVE Mapping:**

| Service | CVE | CVSS | Exploit Type |
|---------|-----|------|-------------|
| vsftpd 2.3.4 | CVE-2011-2523 | 10.0 | Backdoor RCE |
| Samba 3.0.20 | CVE-2007-2447 | 10.0 | RCE via usermap script |
| Apache 2.2 | Multiple | 7.5 | Path traversal / DoS |
| MySQL 5.x | CVE-2012-2122 | 6.5 | Authentication bypass |

#### 💥 Initial Access

vsftpd 2.3.4 backdoor exploited via Metasploit — immediate root-level shell obtained.

#### ⬆️ Privilege Escalation

**Linux (Metasploitable 2) — Key findings via LinPEAS / LSE:**
- Already running as root post-exploit
- Writable `/etc/passwd` and `/etc/shadow` readable
- nmap SUID binary — interactive mode escape to root shell
- MySQL root with no password
- NFS export with `no_root_squash` — entire filesystem mounted

**Windows 7:**
- EternalBlue (MS17-010) → `NT AUTHORITY\SYSTEM`
- Token impersonation via PrintSpoofer

#### 🔒 Persistence

- **Linux:** Attacker SSH public key injected into `/root/.ssh/authorized_keys`
- **Windows:** Scheduled task `updater` running at system start as SYSTEM
```bash
schtasks /create /tn updater /tr "C:\Users\hp\rev.exe" /sc onstart /ru SYSTEM /f
```

#### 🔄 Lateral Movement & Pivoting

- SSH dynamic port forwarding established SOCKS proxy through Metasploitable 2
- Internal discovery via `ip neigh` (Living off the Land)
- Kiwi (in-memory Mimikatz) dumped NTLM hashes — 100% disk-free, AV bypassed
- Pass-the-Hash with `impacket-wmiexec` → admin access to Windows 7 without cracking
```bash
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt --force
```

#### 🏛️ Active Directory Enumeration

```powershell
Get-NetUser        # All domain users
Get-NetComputer    # All domain machines
Get-NetGroup       # Privileged group memberships
```

BloodHound 'Shortest Path to Domain Admin' query revealed multiple viable attack paths — reducing hours of manual analysis to seconds.

#### 🔑 Kerberos Attacks

**LLMNR/NBT-NS Poisoning:**
```bash
responder -I eth0 -dwv
hashcat -m 5600 ntlmv2.txt /usr/share/wordlists/rockyou.txt --force
```

**AS-REP Roasting** — targeted accounts with `DONT_REQ_PREAUTH` flag set:
```bash
GetNPUsers.py lab.local/ -dc-ip <DC-IP> -usersfile users.txt -format hashcat -outputfile hashes.txt
```

**Kerberoasting** — targeted service accounts with registered SPNs:
```bash
impacket-GetUserSPNs lab.local/user:pass -dc-ip <DC-IP> -request -outputfile kerb.txt
hashcat -m 13100 kerb.txt /usr/share/wordlists/rockyou.txt
```

**DCSync + Golden Ticket:**
```bash
mimikatz # lsadump::dcsync /domain:corp.local /user:krbtgt
mimikatz # kerberos::golden /domain:corp.local /sid:S-1-5-21-... /krbtgt:<hash> /user:Administrator /id:500 /ptt
```
Golden Ticket validity: **10 years** — survives password resets until KRBTGT is rotated twice.

---

### Task 03 — Post-Engagement Cleanup

**All persistence artifacts created and removed:**

| Artifact | Location | System | Cleanup Action |
|----------|----------|--------|---------------|
| SSH authorized_key | ~/.ssh/authorized_keys | Linux | Removed attacker public key entry |
| Scheduled Task "updater" | Windows Task Scheduler | Windows 7 | `schtasks /delete /tn updater /f` |
| Meterpreter shell | Process memory | Both | Session terminated — no disk artifact |
| Proxychains SOCKS proxy | Network (port 1080) | Linux pivot | SSH tunnel process killed |
| Golden Ticket | Current session memory | AD Domain | `klist purge` / system reboot |
| DCSync ACL grant | AD ACL on domain object | Domain Controller | `Remove-DomainObjectAcl` |
| Hidden Domain Account | Active Directory OU | Domain Controller | `Remove-ADUser` |
| GPO modification | Group Policy | Domain Controller | Reverted to original backup state |

> **Professional Standard:** Every persistence mechanism must be documented during the engagement. Cleanup is not optional — it is a contractual and ethical obligation.

---

### Task 04 — Final Report

#### Executive Summary

Full Domain Administrator access was achieved. Every system, credential, and confidential resource within the simulated corporate environment was compromised.

**Business Impact:**

| Phase | Impact |
|-------|--------|
| Reconnaissance | Entire network surface mapped in minutes using free tools |
| Initial Access | Backdoored FTP service provided instant root access — no password needed |
| Credential Theft | ALL employee passwords extracted and cracked from one compromised server |
| Lateral Movement | From one breached system, every server in the network was reachable |
| Domain Dominance | A single attacker now controls the entire IT infrastructure permanently |

---

#### 🎯 Risk Rating & MITRE ATT&CK Mapping

| Finding | CVSS | Severity | MITRE ATT&CK |
|---------|------|----------|-------------|
| vsftpd 2.3.4 Backdoor (CVE-2011-2523) | 10.0 | CRITICAL | T1190 — Exploit Public-Facing App |
| Samba Usermap RCE (CVE-2007-2447) | 10.0 | CRITICAL | T1190 — Exploit Public-Facing App |
| EternalBlue MS17-010 | 9.8 | CRITICAL | T1210 — Exploitation of Remote Services |
| Golden Ticket (KRBTGT hash) | 9.8 | CRITICAL | T1558.001 — Golden Ticket |
| DCSync Attack | 9.0 | CRITICAL | T1003.006 — DCSync |
| RCE via cmd= parameter | 9.8 | CRITICAL | T1059 — Command and Scripting Interpreter |
| LLMNR/NBT-NS Poisoning | 8.8 | HIGH | T1557.001 — LLMNR/NBT-NS Poisoning |
| NTLM Hash Extraction (Kiwi/Mimikatz) | 8.5 | HIGH | T1003.001 — LSASS Memory |
| Pass-the-Hash | 8.1 | HIGH | T1550.002 — Pass the Hash |
| ACL Abuse (GenericWrite) | 8.0 | HIGH | T1484.001 — Group Policy Modification |
| AS-REP Roasting | 7.5 | HIGH | T1558.004 — AS-REP Roasting |
| Kerberoasting | 7.5 | HIGH | T1558.003 — Kerberoasting |
| SSH Key Persistence | 7.2 | HIGH | T1098.004 — SSH Authorized Keys |
| Scheduled Task Persistence | 7.2 | HIGH | T1053.005 — Scheduled Task |

---

#### 🛡️ Remediation Recommendations

**Priority 1 — CRITICAL (Immediate Action):**
- Decommission vsftpd 2.3.4, UnrealIRCd, and Samba 3.0.20 — replace with supported versions
- Disable LLMNR and NBT-NS via Group Policy across the entire domain
- Rotate KRBTGT password **TWICE** immediately to invalidate any existing Golden Tickets
- Patch MS17-010 (EternalBlue) on all Windows systems — exploitable since 2017

**Priority 2 — HIGH:**
- Enforce Kerberos pre-authentication for ALL user accounts
- Implement gMSA (Group Managed Service Accounts) for all service accounts
- Audit all AD ACLs using BloodHound — remove unintended permission grants
- Enable LDAP signing and channel binding
- Enforce strong password policies: minimum 15 characters, no common words
- Deploy Microsoft Defender for Identity (MDI) for Kerberos anomaly detection

**Priority 3 — MEDIUM:**
- Implement Tiered Administration Model (Tier 0/1/2 separation)
- Deploy MFA for all privileged accounts and remote access
- Remove unnecessary SUID bits from non-essential binaries (e.g. nmap)
- Implement network segmentation — workstations should not reach servers directly
- Enable Credential Guard on Windows 10/11 to prevent in-memory credential theft

**Priority 4 — STRATEGIC:**
- Schedule quarterly AD security audits (privileged groups, SPNs, ACLs, GPOs)
- Deploy honeypot accounts with fake SPNs and disabled pre-auth for attacker detection
- Implement SIEM alerting on: Event 4768 (TGT requests), Event 4769 (unusual TGS), Event 5136 (AD modifications), DCSync replication events
- Conduct Red Team exercises every 6 months to validate implemented controls

---

## 🛠️ Tools Used

`Nmap` · `Metasploit` · `LinPEAS` · `LSE` · `WinPEAS` · `PowerUp` · `Mimikatz` · `Kiwi` · `Responder` · `Hashcat` · `John the Ripper` · `Impacket` · `BloodHound` · `SharpHound` · `PowerView` · `Proxychains` · `SSH` · `PrintSpoofer` · `WMIExec`

---

## ⚠️ Disclaimer

> All techniques documented in this report were performed exclusively in **authorized, isolated lab environments** using intentionally vulnerable machines (Metasploitable 2, Windows 7 VM, Windows Server Domain Controller). This content is strictly for **educational and research purposes**. Unauthorized use of these techniques against real systems is illegal and unethical.
