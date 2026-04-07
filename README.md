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
* [General Info](#general-information)
* [Technologies Used](#technologies-used)
* [Features](#features)
* [Screenshots](#screenshots)
* [Setup](#setup)
* [Usage](#usage)
* [Project Status](#project-status)
* [Room for Improvement](#room-for-improvement)
* [Acknowledgements](#acknowledgements)
* [Contact](#contact)
<!-- * [License](#license) -->


## Credential Access (TA0006)
- **OS Credential Dumping (T1003)**: Techniques like LSASS memory dumping, `ntds.dit` theft and SAM hive dumping to obtain NTLM hashes or plaintext passwords.
* **Steal or Forge Authentication Certificates (T1649)**: Abuse of AD Certificate Services (AD CS) for persistence or privilege escalation purposes.
- Tools: Mimikatz, Dumpert, Impacket (`secretsdump`), BloodHound.

- Lateral Movement (TA0008):
	- Remote Services (T1021): Using valid accounts to move laterally via SMB/Windows Admin Shares, Remote Desktop Protocol (RDP), or Windows Management Instrumentation (WMI).
	- Pass-the-Hash (PtH) / Pass-the-Ticket (PtT): Utilizing stolen hashes or Kerberos tickets to impersonate users.
- Tools: PsExec, WMIExec, RDP, BloodHound/SharpHound for path analysis.

- Persistence (TA0003):
	- Account Manipulation (T1098): Modifying privileged groups, adding keys to `authorized_keys`, or manipulating user attributes.
	- Golden/Silver Ticket Attacks (T1558): Forging Kerberos Ticket Granting Tickets (TGT) to maintain domain administrator access.
- Tools: Mimikatz, Rubeus, PowerView.

- Privilege Escalation (TA0004):
- Abuse Elevation Control Mechanism (T1548): Leveraging UAC bypass, Token Manipulation, or misconfigured Access Control Lists (ACLs).
* **Tools**: PowerUp, BloodHound. 

- What problem does it (intend to) solve?
- What is the purpose of your project?
- Why did you undertake it?
<!-- You don't have to answer all the questions - just the ones relevant to your project. -->


## Technologies Used
- Tech 1 - version 1.0
- Tech 2 - version 2.0
- Tech 3 - version 3.0


## Features
List the ready features here:
- Awesome feature 1
- Awesome feature 2
- Awesome feature 3


<!-- Optional -->
<!-- ## License -->
<!-- This project is open source and available under the [... License](). -->

<!-- You don't have to include all sections - just the one's relevant to your project -->
