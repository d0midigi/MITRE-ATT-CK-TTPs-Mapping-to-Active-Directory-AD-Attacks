# MITRE ATT&CK Tactics, Techniques & Procedures (TTPs) Mapping to Active Directory (AD) Attacks
> This in-depth guide maps popular and commonly-used Active Directory (AD) attack vectors to the MITRE ATT&CK® Framework. It provides a deep dive into TTPs for credential dumping, lateral movement, persistence, privilege escalation, including specialized AD CS exploitation. Designed for high-impact offensive operations, it features detection rules, attack simulations, and Windows-specific mitigation strategies.

The mapping includes:

* **Toolsets**: Essential software for AD exploitation.
* **Execution**: Exact command-line syntax for rapid deployment.
* **Tradecraft**: Pro-tips, best practices, and common pitfalls for offensive tools.
* **Evasion**: Proven strategies to bypass Blue Team defenses and security controls. 

Focuses on actionable threat intel. Best for red teamss, offensive security professionals, penetration testers, ethical hackers, purple teams, vulnerability specialists, cybersecurity professional.

This resource delivers actionable threat intelligence tailored for Red Teams, penetration testers, ethical hackers, and purple team practitioners looking to harden or exploit AD environments. 

**This repository is intended strictly for educational purposes, designed to assist security professionals, researchers, and ethical hackers in understanding attacker tactics, techniques, and procedures (TTPs) to strengthen defensive strategies. The authors are not liable for any misuse of this information.**
 
## Table of Contents
* [Credential Access (TA0006)](#credential-access)
* [Lateral Movement (TA0008)](#lateral-movement)
* [Persistence (TA0003)](#persistence)
* [Privilege Escalation (TA0004)](#privilege-escalation)
* [Actionable Mitigation and Defense Strategies](#actionable-mitigation-and-defense-strategies)
* [Monitoring (Detection)](#monitoring-detection)
* [Offensive Security Best Practices & Blue Team Advice](#offensive-security-best-practices-and-blue-team-advice)
* [Core Active Directory TTPs](#core-active-directory-ttps)
* [Discovery](#discovery)
* [Lateral Movement & Persistence](#lateral-movement-and-persistence)
* [Active Directory CS (Certificate Services)](#active-directory-cs-certificate-services)
* [Delegation & Relationship Attacks](#delegation-and-relationship-attacks)
* [Manipulation & Impersonation](#manipulation-and-impersonation)
* [Network & Protocol Attacks (In-Network)](#network-and-protocol-attacks-in-network)
* [Red Team Execution: Path to Domain Admin](#red-team-execution-path-to-domain-admin)
* [Technical Deep-Dive Into Evasion and Tool Command Syntax](#technical-deep-dive-into-evasion-and-tool-command-syntax)

<!-- * [License](#license) -->


## Credential Access [(TA0006)](https://attack.mitre.org/tactics/TA0006/)
* **OS Credential Dumping** [(T1003)](https://attack.mitre.org/techniques/T1003/003/): OS Credential Dumping (MITRE ATT&CK Technique T1003) is a, if not the, primary method threat actors use to transition from initial system access to full network compromise. By stealing password hashes or plaintext credentials stored in operating system memory or databases, attackers can escalate privileges and move laterally across an environment. This technique is frequently used by ransomware gangs and APT groups.
**TL;DR**: Techniques like LSASS memory dumping, `ntds.dit` theft and SAM hive dumping to obtain NTLM hashes or plaintext passwords.
* **Steal or Forge Authentication Certificates (T1649)**: Steal or Forge Authentication Certificates (T1649) is a critical MITRE ATT&CK technique focusing on abusing Active Directory Certificate Services (AD CS) to gain unauthorized access. Because AD CS enables certificate-based authentication (using certificates instead of passwords), compromising the certificate infrastructure allows attackers to bypass traditional password-based security controls, establish long-term persistence, and escalate privileges to the highest levels (e.g., Domain Admin). 
**TL;DR**: Abuse of AD Certificate Services (AD CS) for persistence or privilege escalation purposes.
### Tools
* **Mimikatz**: Mimikatz is a powerful open-source post-exploitation tool designed to extract plain-text passwords, hashes, PINs, and Kerberos tickets directly from Windows memory (LSASS process). It is primarily used by attackers to escalate privileges, move laterally through networks, and create persistent access through techniques like pass-the-hash and Golden Ticket attacks.
#### Key Capabilities and Functions:
* **Credential Dumping (`sekurlsa`)**: Extracts credentials from memory, including user passwords in plain text, NTLM hashes, and Kerberos tickets.
* **Pass-the-Hash (PtH)**: Uses stolen NTLM hashes to authenticate as a user without needing the original password.
* **Pass-the-Ticket/Golden Ticket**: Generates forged Kerberos tickets to gain unauthorized domain access and impersonate any user, often indefinitely.
* **Privilege Escalation**: Elevates low-level user access to administrator or SYSTEM-level privileges.
* **Certificate Export**: Exports certificates and private keys from the Windows Certificate Store, even if marked as non-exportable.
* **Fileless Execution**: Often executed in memory via PowerShell or similar methods to avoid detection on disk.
* **Dumpert**: Dumpert is an open-source, specialized security tool designed to create a memory dump of the Local Security Authority Subsystem Service (LSASS) process on Windows operating systems. It is primarily used during red team engagements and penetration tests to steal credentials, such as password hashes and Kerberos tickets, while avoiding detection by antivirus (AV) and Endpoint Detection and Response (EDR) solutions. 
#### Key Functions and Capabilities
* **LSASS Memory Dumping**: It extracts the memory of lsass.exe, where sensitive user credentials are stored, saving them into a `.dmp` file.
* **AV/EDR Evasion*: Unlike standard tools (like ProcDump or Mimikatz), Dumpert avoids using common Windows API functions that are heavily monitored by security software.
* **Direct System Calls (Syscalls)**: It bypasses user-mode hooks by executing system calls directly to the kernel, making it difficult for security products to detect the activity.
* **API Unhooking**: The tool unhooks API functions to further mask its activities.
* **Fileless Operation (sRDI)**: It provides an sRDI (shellcode Reflective DLL Injection) version, allowing it to be injected directly into memory via Cobalt Strike, thus avoiding writing files to the disk. 
#### Usage Context
* **Credential Theft**: Once the lsass.dmp file is created, it can be analyzed offline using tools like Mimikatz or Pypykatz to extract plaintext passwords and hashes.
* **Lateral Movement**: The stolen credentials enable attackers to move laterally across a network.
* **Post-Exploitation**: It is used after gaining elevated privileges (SYSTEM) on a compromised system.
* **Impacket (`secretsdump`)**: Impacket's secretsdump.py is a powerful, open-source Python script used to remotely or locally extract sensitive secrets—such as user password hashes and credentials—from Windows systems without installing an agent on the target machine. It is widely used by penetration testers for security auditing and by attackers for lateral movement and privilege escalation.
#### Core Functionalities
* **`secretsdump`**: Performs various techniques to dump secrets, including: 
* **SAM and LSA Secrets Extraction**: It reads the Security Account Manager (SAM) and Local Security Authority (LSA) registry hives to obtain local user NTLM hashes, cleartext credentials, and cached domain credentials.
* **NTDS.dit Extraction**: It extracts the Active Directory database (NTDS.dit) from Domain Controllers, allowing for the retrieval of NTLM hashes, Kerberos keys, and usernames for all domain users.
* **DCSync Attack**: It utilizes the DRSR (Directory Replication Service Remote) protocol to mimic a Domain Controller and pull password hashes, a common method used to dump domain credentials without executing code on the DC.
* **Offline Hive Parsing**: It can parse SAM, SECURITY, and SYSTEM registry hives that have already been dumped and saved locally.
* **Volumen Shadow Copy (VSS)**: If files are locked, it uses Volume Shadow Copies to read locked database files. 
#### Key Features for Attackers/Pentester
* **Agentless**: No agent or binary is dropped on the target machine, which helps evade detection.
* **Multiple Authentication Methods**: Supports authentication via username/password, NTLM hashes (Pass-the-Hash), or Kerberos keys (Pass-the-Ticket).
* **Service Manipulation**: If required, it can remotely enable the Remote Registry service to extract credentials and restore it to its original state afterward. 
#### Typical Use Case
An attacker with local administrator privileges on one machine can use secretsdump to connect to a domain controller or another machine, extract hashes, and then use those hashes to authenticate to other systems in the network (lateral movement).
* **BloodHound**: BloodHound is an open-source cybersecurity tool that uses graph theory to map and analyze relationships within Active Directory (AD) and Azure environments. It identifies hidden attack paths, privilege escalations, and misconfigurations, allowing red teams to move laterally and blue teams to remediate security risks. 
#### Key Functions of BloodHound:
* **Attack Path Visualization**: It maps relationships between users, groups, computers, and permissions, visualizing complex AD environments.
* **Privilege Escalation Mapping**: It finds the shortest, most efficient path from a compromised low-privilege user to high-privilege targets, such as Domain Admins.
* **Data Collection (SharpHound/AzureHound)**: It uses ingestors (SharpHound for AD, AzureHound for Azure) to collect data on user sessions, group memberships, and ACLs.
* **Defensive Analysis**: Defenders (blue teams) use it to identify and eliminate dangerous privilege relationships and misconfigurations.
* **Attack Simulation**: Red teams use it to plan lateral movement and simulate ransomware-style attacks. 
#### Components
* **Data Ingestors**: SharpHound.exe (C#) or PowerShell scripts gather network data.
* **Graph Database**: Neo4j stores the relationships.
* **Visualization GUI**: A web interface (Community Edition) or legacy app displays the graph, revealing attack paths. 
BloodHound is available as an open-source Community Edition and a managed enterprise version. 
---
## Lateral Movement (TA0008)
Lateral Movement (TA0008) in the MITRE ATT&CK framework refers to the techniques cyber adversaries use to navigate through a network, moving from an initially compromised system to other hosts, to expand access, steal data, or deploy ransomware. It is a critical post-compromise stage, often involving stolen credentials and legitimate tools to evade detection. 
### Key Aspects and Techniques
Attackers use several methods to move laterally, often blending in with authorized user activity: 
* **Remote Services (T1021)**: Utilizing valid credentials to log in remotely via RDP, SSH, or VPN.
* **Lateral Tool Transfer (T1570)**: Moving tools or malware between systems to further the attack.
* **Pass-the-Hash (T1550.002)**: Using stolen password hashes to authenticate, bypassing the need for a plaintext password.
* **Exploitation of Remote Services (T1210)**: Exploiting vulnerabilities in network services to gain unauthorized access.
* **Internal Spearphishing (T1534)**: Using a compromised internal account to send phishing emails to other employees within the same organization.
* **Windows Admin Shares (T1021.002)*: Utilizing built-in network shares to copy files and move between systems. 
### Common Usage Examples
* **RDP/VPN Hijacking**: An attacker uses stolen VPN credentials to log into a workstation, then uses RDP to move to a sensitive server.
* **Using Admin Tools**: Utilizing Windows Management Instrumentation (WMI) or PowerShell Remoting to execute code on remote machines.
* **Cloud Lateral Movement**: Accessing shared cloud resources by stealing API keys or leveraging misconfigured IAM roles to jump between cloud tenants. 
### Imprtant Note About Lateral Movement
This tactic is essential for threat actors to achieve their final objectives, such as stealing intellectual property, escalating privileges, or gaining persistence across an entire network. It is heavily used in approximately 60% of attacks, including ransomware campaigns. 
* **Remote Services (T1021)**: Remote Services involve threat actors using legitimate credentials to move laterally within a network by abusing built-in administrative tools like RDP, SMB/Admin Shares, and WMI. By blending into normal administrative activity (living off the land), attackers can steal data, install malware, or pivot to new targets. 
### Key Aspects of Remote Services Lateral Movement:
* **SMB/Windows Admin Shares (T1021.002)**: Attackers leverage valid accounts to connect to default administrative shares (e.g., `C$`, `ADMIN$`) to copy malicious tools and move laterally.
* **Remote Desktop Protocol (T1021.001)**: Attackers use RDP to log into systems interactively with a graphical user interface, often using stolen credentials, allowing them to act as a normal user.
* **Windows Management Instrumentation (WMI) (T1021.003)**: Attackers use WMI to execute code remotely on other Windows systems, often used for stealthy command execution, bypassing standard monitoring.
* **Other Channels**: This technique also covers abuse of SSH, VNC, Windows Remote Management, and Cloud Services to access remote infrastructure. 
### Detection and Mitigation Strategies:
* **Monitor Activity**: Look for unusual logon activity, such as, for example, user accounts accessing multiple systems in a short time, or RDP/SMB activity outside of normal working hours.
* **Restrict Access**: Use Windows Firewall to restrict SMB access, disable administrative shares if not needed, and restrict local administrator accounts to specific workstations.
* **Credential Management**: Enforce strong, unique passwords for local admin accounts to prevent pass-the-hash attacks.
* **TL;DR**: Using valid accounts to move laterally via SMB/Windows Admin Shares, Remote Desktop Protocol (RDP), or Windows Management Instrumentation (WMI).
* **Pass-the-Hash (PtH) / Pass-the-Ticket (PtT)**: Utilizing stolen hashes or Kerberos tickets to impersonate users.
### Tools
* PsExec
* WMIExec
* RDP
* BloodHound/SharpHound for path analysis

## Persistence (TA0003)
* **Account Manipulation (T1098)**: Modifying privileged groups, adding keys to `authorized_keys`, or manipulating user attributes.
* **Golden/Silver Ticket Attacks (T1558)**: Forging Kerberos Ticket Granting Tickets (TGT) to maintain domain administrator access.
### Tools
* Mimikatz
* Rubeus
* PowerView

## Privilege Escalation (TA0004)
* **Abuse Elevation Control Mechanism (T1548)**: Leveraging UAC bypass, Token Manipulation, or misconfigured Access Control Lists (ACLs).
### Tools
* PowerUp
* BloodHound

## Actionable Mitigation and Defense Strategies
### Hardening (Prevention)
* **Disable Legacy Protocols**: Disable SMBv1, LLMNR, and restrict NTLMv1.
* **Credential Protection**: Implement Credential Guard, LAPS (Local Administrator Password Solution) for local admin accounts.
* **Tiered Administration**: Implement a Tiered Admin Model to restrict high-privilege accounts from lower-tier workstations.
* **AD CS Security**: Audit and harden AD Certificate Services to prevent forgery.

## Monitoring (Detection)
* **SIEM/Audit Logs**: Monitor for Event ID 4624 (successful login), 4688 (process creation), and 4720-4730 (user/group management).
* **Detection Rules**: Create alerts for `ntds.dit` file access, unusual service creation (`PsExec`), and modification of sensitive group memberships (e.g., Domain Admins).
* **Simulation**: Use tools like Caldera or Infection Monkey to simulate techniques like DCSync (T1003.006) to test detection efficacy. 

## Offensive Security Best Practices & Blue Team Advice
### Tools Do's and Don'ts
* **Do**: Use legitimate administrative tools (Living off the Land) to avoid signature detection (e.g., PowerShell, WMI).
* **Don't**: Run tools like Mimikatz without renaming or obfuscating the executable, as it is highly signatured.
### Avoiding Blue Team Obstacles:
* **Operational Security (OPSEC)**: Avoid noisy scans. Use SharpHound with specific collection methods rather than All.
* **Living off the Land**: Use `net.exe`, `dsquery`, or native PowerShell cmdlets to reduce the forensic footprint.
* **Targeted Enumeration**: Use BloodHound to identify the shortest, least noisy attack path to Domain Admin, rather than trying to attack every object. 

## Core Active Directory TTPs
Active Directory attacks typically span four major Tactics: Credential Access, Discovery, Lateral Movement, and Persistence.
### Credential Access
* **Kerberoasting (T1558.003)**: Requesting Service Tickets (TGS) for service accounts and cracking them offline.
* **AS-REP Roasting (T1558.004)**: Targeting accounts with 'Do not require Kerberos pre-authentication" enabled to crack their passwords.
* **`NTDS.dit` Dumping (T1003.003)**: Stealing the primary AD database file from a Domain Controller to extract all user hashes.
* **Pass-the-Hash (PtH) (T1550.002)**: Using a captured NTLM hash to authenticate without needing the plaintext password.
* **Pass-the-Ticket (PtT) (T1550.003)**: Using captured Kerberos tickets (TGT/TGS) to move across the domain.

## Discovery
* **Domain Trust Discovery (T1482)**: Using tools like AdFind or BloodHound to map relationships between domains and forests.
* **Account Discovery (T1087.002)**: Enumerating domain accounts via LDAP queries or native tools like `net user /domain`.
* **Permission Groups Discovery (T1069.002)**: Identifying high-value groups like Domain Admins or Enterprise Admins.
* **Domain Controller Discovery (T1018)**: Locating Domain Controllers via DNS or `nltest`.

## Lateral Movement & Persistence
* **Golden Ticket (T1558.001)**: Forging a Ticket Granting Ticket (TGT) using the `krbtgt` account hash for permanent domain-wide access.
* **Silver Ticket (T1558.002)**: Forging service tickets to gain unauthorized access to specific services (e.g., MSSQL, CIFS).
* **DCShadow (T1207)**: Registering a rogue Domain Controller to inject malicious objects or change permissions.
* **DCSync (T1003.006)**: Impersonating a Domain Controller to request account hashes from another DC via replication protocols.

## Active Directory CS (Certificate Services)
Exploiting AD CS is currently one of the most effective ways to escalate privileges from a standard user to Domain Admin.
* **ADCS ESC1/ESC2/ESC3 (T1649)**: Misconfigured Certificate Templates.
Requesting a certificate for a high-privileged user (like a Domain Admin) using a low-privileged account.
* **AD CS ESC8 (T1557.001)**: NTLM Relay to HTTP Enrollment options.
Coercing a Domain Controller to authenticate to an attacker machine, then relaying that to the AD CS web interface to get a DC certificate.
* **Certificate Theft (T1552.004)**: Exporting private keys and certificates from user stores to bypass MFA or impersonate identities.

## Delegation & Relationship Attacks
These attacks leverage the 'intended" logic of AD to gain unauthorized access.
* **Unconstrained Delegation (T1558)**: Compromising a server where Unconstrained Delegation is enabled. When a high-privileged user connects, their TGT is stored in memory and can be stolen and cracked offline.
* **Constrained Delegation (T1558.003)**: Abusing the `msDS-AllowedToDelegateTo` attribute to imp any user to a specific service.
* **Resource-Based Constrained Delegation (RBCD) (T1558)**: Configuring a target computer's `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute to allow an attacker-controlled machine to impersonate users to it.
* **Group Policy Object (GPO) Abuse (T1484.001)**: Modifying a GPO to push a scheduled task, malicious script, or new local admin user to all workstations in an OU.

## Manipulation & Impersonation
* **SAM Name Spoofing / NoPac (T1558)**: Exploiting logic flaws (CVE-2012-42278/42287) where a machine account name is changed to match a Domain Controller, tricking the KDC into issuing a high-privileged TGT.
* **ACL/ACE Modification (T1098)**: Adding `GenericAll` or `WriteDacl` permissions to a target object (like a User or Group) to ensure long-term control.
* **`AdminSDHolder` (T1098)**: Modifying the permissions of the `AdminSDHolder` container. AD automatically propagates these permissions to protected groups (like Domain Admins) every 60 minutes.

## Network & Protocol Attacks (In-Network)
* **LLMNR/NBT-NS Poisoning (T1557.001)**: Using tools like Responder to spoof name resolution responses and capture NTLMv2 hashes from the network.
* **IPv6 DNS Takeover / `mitm6` (T1557.001)**: Acting as a rogue IPv6 DNS server to force Windows machines to authenticate via WPAD, allowing for NTLM relaying.
* **Remote Service Creation (T1543.003)**: Using `PsExec` or `sc.exe` to create services on remote machines once local admin rights are obtained. 

## Red Team Execution: Path to Domain Admin

### 1. Initial Access & Credential Capture
#### TTPs:
* LLMNR/NBT-NS Poisoning (T1557.001)
* Spearphishing (T1566.001)
#### Primary Tools:
* **Responder**: Captures NTLMv1/v2 hashes via protocol poisoning.
* **Inveigh**: A PowerShell/C# version of Responder (better for Windows-only environments).
* **Evilginx2**: For MFA-bypass phishing.
#### Evasive Techniques:
* **Analyze Network Traffic First***: Run Responder in "Analyze" mode (`-A`) to identify noise before poisoning.
* **Targeted Poisoning**: Use the `-I` flag to target specific IP ranges rather than the targeting the entire subnet which will set off detection alarms.
* **Socket Selection**: Use `Inveigh` to avoid opening new high-risk ports that EDR monitors.

### 2. Reconnaissance
#### TTPs:
* Domain Trust / Object Discovery (T1482)
#### Primary Tools:
* **BloodHound / SharpHound**: Maps the "Six Degrees of Domain Admin."
* **AdFind**: Command-line tool for specific LDAP queries.
* **Snaffler**: Finds sensitive data in AD (passwords, certificates) in open shares.
#### Evasive Techniques:
* **Stealthy Collection**: Use `SharpHound` with the `--collectionmethod DCOnly` or `LogonSessions` to avoid touching every workstation.
* **Slow & Steady**: Use the `--throttling` and `--jitter` flags to bypass behavior-based detection of massive LDAP queries.
* **DON'T USE `.exe`**: Run SharpHound via In-Memory Reflection using `execute-assembly` in your C2 (Cobalt Strike, or Havoc).

### 3. Privilege Escalation
#### TTPs:
* Kerberoasting (T1558.003)
* AD CS ESC1 (T1649)
### Primary Tools:
* **Rubeus**: The industry standard for AD Kerberos interactions.
* **Certipy / Certify**: For identifying and exploiting AD CS misconfigurations.
* **Impacket Tool Suite's (`GetUserSPNs.py`)**: For remote Kerberoasting from a *nix attack box.
#### Evasive Techniques:
* **Targeted Roasting**: **Do not roast every SPN**. Only request tickets for accounts with high-privilege keywords (e.g., `*adm*`, `*svc*`, `*sql*`).
* **Encryption Downgrading**: Avoid forcing weak encryption (RC4) if the targeted domain supports AES, or RC-4 only requests are high-fidelity alerts.
* **OPSEC-Safe AD CS**: Use `Certify` to find templates but avoid the `/enforce` flag unless absolutely necessary to prevent accidental lockouts or alerts from triggering.

### 4. Lateral Movement
#### TTPs:
* Overpass-the-Hash / Pass-the-Ticket (PtT)
### Primary Tools:
* **Mimikatz**: The classic tool for credential extraction and injection.
* **Impacket Tool Suite (`wmiexec.py` / `PSExec.py`)**: For executing commands on remote targets.
* **Evil-WinRM**: For remote management access using captured credentials.
#### Evasive Techniques:
* **Avoid Mimikatz on Disk**: **Always run it via memory** or use LSASS Minidumps (e.g., `comsvcs.dll` via `rundll32`) to avoid signature detection.
* **WMI over SMB**: Use `wmiexec` or `dcomexec` instead of `psexec`, as `psexec` creates a visible service that EDRs will flag instantly.
* **SOCKS Proxying**: Use `Chisel` or `Ligolo-ng` to tunnel Impacket traffic through your C2 agent to appear as inconspicuous local network traffic.

### 5. Persistence
#### TTPs:
* Golden Ticket (T1558.001)
* GPO Modification (T1484.001)
#### Primary Tools:
* **Mimikatz (`lsadump::lsa /patch`)**: To extract the `krbtgt` hash.
* **`SharpGPOAbuse`**: For adding malicious tasks or rights to GPOs.
* **`PyGPOTools`**: For remote GPO abuse and manipulation.
#### Evasive Techniques:
* **`krbtgt` Rotation**: If you create a Golden Ticket, ensure its lifetime is short (e.g., 10 hours) to match normal domain policy and avoid setting off any file mismatch alerts.
* **GPO "Hidden" Tasks**: When modifying GPOs, use an existing, legitimate GPO and add a "hidden" scheduled task that only triggers on specific conditions.
* **DCSync Timing**: When running `DCSync` to get the `krbtgt` hash, target a specific DC that is known for higher traffic to blend in with replication and background 'network noise.'

### Quick Red Team Tool Breakdown
| Attack Phase | Tool(s) | MITRE ID |
| -------- | -------- | -------- |
| Initial Capture    | [Responder](https://www.kali.org/tools/responder/), [Inveigh](https://github.com/kevin-robertson/inveigh)     | [T1557.001](https://attack.mitre.org/techniques/T1557/001/)     
| Mapping    | [BloodHound](https://www.kali.org/tools/bloodhound)/, [SharpHound](https://github.com/SpecterOps/SharpHound)     | [T1482](https://attack.mitre.org/techniques/T1482/)     
| Kerberos Exploits    | [Rubeus](https://www.kali.org/tools/rubeus/), [Impacket](https://www.kali.org/tools/impacket/)     | [T1558](https://attack.mitre.org/techniques/T1558/)     
| AD CS Exploits    | [Certipy](https://github.com/ly4k/Certipy), [Certify](https://github.com/GhostPack/Certify)     | [T1649](https://attack.mitre.org/techniques/T1649/)     |
| Movement    | [Mimikatz](https://www.kali.org/tools/mimikatz/), [WMIexec](https://github.com/XiaoliChan/wmiexec-Pro)     | [T1550](https://attack.mitre.org/techniques/T1550/)     |
| Persistence    | [SharpGPOAbuse](https://github.com/byronkg/SharpGPOAbuse), [Mimikatz](https://www.kali.org/tools/mimikatz/)     | [T1484](https://attack.mitre.org/techniques/T1484/)    

### 6. Coerced Authentication & NTLM Relaying
These techniques allow an individual to move laterally and escalate privileges without ever needing to crack a password or find a plaintext credential.
### TTPs:
* **Adversary-in-the-Middle (AitM) (T1557.001)**: Adversary-in-the-Middle (AitM) is a specific cybersecurity sub-technique within the MITRE ATT&CK framework where an attacker poisons themselves between a user and a legitimate network resource to intercept and manipulates authentication traffic. By exploiting weak network protocols (LLMNR, NBT-NS, mDNS), attackers are able to trick systems into sending authentication hashes to the attacker-controlled machine, which are then relayed to other services to gain unauthorized access.
* **Exploitation of Remote Services (T1210)**: This involves forcing a high-privileged machine (like a Domain Controller) to authenticate to you, then "relaying" that session to a target (like AD CS or a sensitive, mission-critical server).
### Primary Tools:
* **PetitPotam/SpoolSample**: Coerces a machine account to authenticate via MS-EFSR or Print Spooler.
* **Impacket Tool Suite (`ntlmrelayx`)**: The "engine" that catches the authentication and relays it to LDAP, SMB, or HTTP.
* **PrinterBug**: Specifically targets the Print System Remote Protocol to force authentication.
### Evasive Techniques:
* **Relay to LDAP/S**: Many EDRs monitor SMB relaying. Relaying NTLM to LDAP (to modify permissions) or AD CS (to get a certificate) is often less scrutinized.
* **Multi-Relay**: Use `--multi-relay` in `ntlmrelayx` to maintain a persistent relay session across multiple targets simultaneously.

### 7. Shadow Credentials (Whiteshadow)
### TTPs:
* **Account Manipulation (T1098)**: If you have "`GenericWrite`" or "`WriteProperty`" over a user or computer object, you can add a public key to their `msDS-KeyCredentialLink` attribute and then authenticate as them via PKINIT.
### Primary Tools:
* **Whisker / `pyWhisker`**: Specifically designed to manipulate the `msDS-KeyCredentialLink` attribute.
* **Rubeus**: Used to perform the subsequent PKINIT authentication to get a TGT.
### Evasive Techniques:
* **Attribute Cleanup**: This technique leaves a permanent attribute on the object. Always remove the key link immediately after obtaining your TGT to minimize the forensic footprint.
* **Targeting "Dead" Accounts**: Apply shadow credentials to accounts that are rarely used but have high permissions to avoid triggering "active session" alerts.

### 8. Local Privilege Escalation (LPE) to Domain Access
### TTPs:
* **Access Token Manipulation (T1134)**: Access Token Manipulation (T1134) is a MITRE ATT&CK technique used by adversaries to gain unauthorized access to systems, escalate privileges, and evade security controls by manipulating the Windows security tokens that are associated with processes and threads. This technique primarily targets Windows environments, allowing an attacker to change the security context of a running process to appear as if it is running as a different user (e.g., Administrator or `NT AUTHORITY\SYSTEM`).
* **Boot or Logon Autostart Execution (T1547)**: Boot or Logon AutoStart Execution (T1547) is a MITRE ATT&CK technique where attackers configure malicious programs to run automatically when a system boots or a user logs in. By abusing system startup mechanisms like registry keys, startup folders, or `systemd`, they achieve persistence, privilege escalation, and stealth.
### Primary Tools:
* **SharpUp / Seatbelt**: Audits for local misconfigurations (unquoted service paths, modifiable binaries).
* **GodPotato / PetitPotato**: Modern "Potato" exploits to escalate from Service Accounts to `SYSTEM`.
* **Mimikatz (`sekurlsa::tickets`)**: To "harvest" tickets of users currently or previously logged into the machine.
### Evasive Techniques:
* **Process Injection**: Instead of running an exploit as a new process, inject your LPE code into a trusted process like `svchost.exe` or `spoolsv.exe`.
* **Token Impersonation**: Use `incognito` (via Metasploit/Cobalt Strike) to steal tokens from memory without dumping LSASS, which is a massive EDR trigger.

### 9. The "Golden GMSA" Attack
### TTPs: Steal or Forge Kerberos Tickets (T1558)
Group Managed Service Accounts (gMSAs) are often used for high-privilege services. If you can read the `msDS-ManagedPassword` attribute, you own the service.
### Primary Tools:
* **`GMSAPasswordReader`**: Specifically for extracting gMSA passwords.
* **BloodHound**: To identify which users have the rights to read gMSA passwords.
### Evasive Techniques:
* **LDAP Filtering**: Use targeted LDAP filters to read only the specific gMSA attribute you need, rather than querying the entire object, which might trigger behavioral alerts.
### Red Team TTP Mapping Table
| Attack Phase | Technique(s) | MITRE TTP | Tool(s) |
| -------- | -------- | -------- | -------- |
| Initial     | Coerced Authentication     | T1210    | PetitPotam     | 
| Escalation     | Shadow Credentials     | T1098    | Whisker     |
| Recon     | gMSA Discovery     | T1087.002    | BloodHound     |
| Movement     | Token Impersonation     | T1134    | Incognito     |
| Persistence     | Golden Certificate     | T1558.001    | ForgeCert     |

## Technical Deep-Dive Into Evasion and Tool Command Syntax
### 1. Detection Evasion Deep-Dive (AMSI & ETW)
Modern EDRs rely on AMSI to scan memory and ETW to log suspicious API calls (like OpenProcess on LSASS).
Bypassing AMSI (Antimalware Scan Interface):
* **Patching**: Overwrite the AmsiScanBuffer function in `amsi.dll` with a "return" (`0xCB`) so it always returns "Clean."
* **Obfuscation**: Use tools like Chimera or `Invoke-Obfuscation` to change variable names and string types in PowerShell scripts.
* **Bypassing ETW (Event Tracing for Windows) - ETW Patching**: Similar to AMSI, you can patch `EtwEventWrite` in `ntdll.dll` to prevent the OS from sending telemetry to the EDR.
* **Unhooking**: Use `LdrBuiltin` or SharpBlock to load a "fresh" copy of ntdll.dll from disk, effectively removing the EDR's hooks.
<!-- Optional -->
<!-- ## License -->
<!-- This project is open source and available under the [... License](). -->

<!-- You don't have to include all sections - just the one's relevant to your project -->
