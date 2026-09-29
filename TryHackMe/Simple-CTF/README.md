# TryHackMe – Simple CTF | Penetration Testing Report

> ⚠️ **Legal Disclaimer:** This report documents a penetration test performed against an authorized TryHackMe lab machine (*Simple CTF*) in an isolated training environment. All techniques were performed only against the designated practice target with permission. Do not use these techniques against systems without prior authorization.

---

## 📌 Overview

This repository contains my penetration testing report for the **TryHackMe – Simple CTF** room.

The assessment followed a practical penetration testing workflow:

**Reconnaissance → Enumeration → Credential Discovery → Initial Access → Privilege Escalation → Post-Exploitation**

The lab demonstrated how multiple security weaknesses can be chained together to obtain privileged access to a target system.

### Lab Information

| Attribute             | Details                   |
| --------------------- | ------------------------- |
| **Platform**          | TryHackMe                 |
| **Room**              | Simple CTF (`easyctf`)    |
| **Target IP**         | `10.49.143.130`           |
| **Attacking Machine** | Kali Linux                |
| **Target OS**         | Ubuntu 16.04.6 LTS        |
| **Assessment Type**   | Authorized Lab / CTF      |
| **Access Achieved**   | Root                      |
| **Flags Captured**    | `user.txt` and `root.txt` |

---

## 🎯 Objective

The objective of this lab was to identify exposed services, enumerate the target, obtain initial access, escalate privileges, and capture the required flags.

---

## 🛠️ Tools Used

| Tool         | Purpose                                      |
| ------------ | -------------------------------------------- |
| **Nmap**     | Port and service enumeration                 |
| **FTP**      | Anonymous FTP enumeration and file retrieval |
| **Hydra**    | Credential auditing against SSH              |
| **SSH**      | Remote access                                |
| **GTFOBins** | Privilege-escalation technique research      |

---

## 🧭 Methodology

The assessment followed these stages:

1. **Reconnaissance** – Identify open ports and running services.
2. **Enumeration** – Investigate exposed FTP, HTTP and SSH services.
3. **Credential Discovery** – Identify and validate weak credentials.
4. **Initial Access** – Establish an SSH session.
5. **Privilege Escalation** – Analyze `sudo` permissions.
6. **Post-Exploitation** – Obtain root access and capture the flags.

---

# 1. Reconnaissance – Nmap Enumeration

### Command

```bash
nmap -sV 10.49.143.130
```

### Results

| Port       | Service | Version             |
| ---------- | ------- | ------------------- |
| `21/tcp`   | FTP     | vsftpd 3.0.3        |
| `80/tcp`   | HTTP    | Apache httpd 2.4.18 |
| `2222/tcp` | SSH     | OpenSSH 7.2p2       |

### Observation

Three exposed services were identified:

* FTP on port `21`
* HTTP on port `80`
* SSH on non-standard port `2222`

FTP was investigated further because anonymous access could potentially expose files or useful information.

---

# 2. FTP Enumeration

### Command

```bash
ftp 10.49.143.130
```

The FTP service permitted **anonymous authentication**.

After accessing the `pub` directory, the following file was discovered:

```text
ForMitch.txt
```

The file was downloaded for analysis.

### Observation

The file contained information indicating that the system user's password was weak and reused.

This information provided a useful lead for further credential auditing.

### Security Issue

**Anonymous FTP access** can expose files to unauthenticated users and may lead to information disclosure.

---

# 3. Credential Discovery – SSH

Based on the information obtained during FTP enumeration, the username `mitch` was investigated against the SSH service.

### Command

```bash
hydra -l mitch -P /usr/share/wordlists/rockyou.txt ssh://10.49.143.130:2222
```

### Result

A valid SSH credential was identified during the authorized TryHackMe lab.

> 🔒 **Credential note:** The recovered password is intentionally omitted from this public portfolio README.

### Observation

The weak password allowed authenticated SSH access to the target.

---

# 4. Initial Access via SSH

### Command

```bash
ssh mitch@10.49.143.130 -p 2222
```

After authentication, the current user was verified:

```bash
whoami
```

The result confirmed access as:

```text
mitch
```

---

## 🔎 Sudo Enumeration

The user's available sudo privileges were then examined:

```bash
sudo -l
```

The configuration showed that `mitch` could execute `/usr/bin/vim` as root without requiring a password.

### Security Impact

Allowing a user to execute a shell-capable program such as Vim with unrestricted root privileges can provide a direct path to privilege escalation.

---

# 5. Privilege Escalation Research

The `sudo` configuration was investigated using **GTFOBins** as a reference for understanding the security implications of allowing Vim to run with elevated privileges.

The relevant technique demonstrates that a shell-capable editor can be abused when incorrectly granted unrestricted `sudo` privileges.

### Reference

[GTFOBins – Vim](https://gtfobins.org/gtfobins/vi/)

---

# 6. Privilege Escalation & Flag Capture

The vulnerable `sudo` configuration was used within the authorized TryHackMe environment to obtain root-level access.

After privilege escalation, access was verified with:

```bash
whoami
```

The result confirmed:

```text
root
```

The root flag was then retrieved from the root user's directory.

### Flags

| Flag       | Status     |
| ---------- | ---------- |
| `user.txt` | ✅ Captured |
| `root.txt` | ✅ Captured |

> 🚩 Flag values are intentionally omitted from this public README. The complete evidence and screenshots are available in the accompanying PDF report.

---

# 🔍 Findings Summary

| # | Finding                          | Severity | Security Impact                                         |
| - | -------------------------------- | -------- | ------------------------------------------------------- |
| 1 | Anonymous FTP access             | High     | Allowed unauthenticated access to exposed files         |
| 2 | Sensitive information disclosure | Critical | Exposed information that assisted credential discovery  |
| 3 | Weak SSH password                | Critical | Allowed credential compromise through password auditing |
| 4 | Excessive `sudo` privileges      | Critical | Allowed a user to execute Vim with root privileges      |
| 5 | Insecure `sudo` configuration    | Critical | Enabled a direct privilege-escalation path              |

> **Severity ratings above are for the vulnerabilities observed in this TryHackMe training lab and should not be interpreted as a production security assessment.**

---

# 🛡️ Recommendations

### 1. Disable Anonymous FTP

Anonymous FTP access should be disabled unless there is a clearly defined and controlled business requirement.

### 2. Remove Sensitive Information

Do not store credentials, password hints, or other sensitive information in publicly accessible directories.

### 3. Enforce Strong Passwords

Use strong, unique passwords and prevent password reuse.

For SSH access, key-based authentication should be preferred where appropriate.

### 4. Apply Least Privilege

Users should only receive the minimum `sudo` permissions required for their role.

Avoid granting unrestricted `sudo` access to programs that can spawn shells or execute arbitrary commands.

### 5. Audit Sudo Configuration

Regularly review:

```bash
sudo -l
```

and the relevant `sudoers` configuration to identify excessive privileges.

### 6. Monitor SSH Authentication

Centralized logging and alerting can help identify repeated failed SSH authentication attempts and potential credential attacks.

---

# 🧠 Skills Demonstrated

This lab helped reinforce practical cybersecurity skills in:

* Network reconnaissance
* Nmap service enumeration
* FTP enumeration
* Anonymous FTP access analysis
* Credential auditing
* SSH enumeration and access
* Linux privilege escalation
* `sudo` misconfiguration analysis
* GTFOBins research
* Post-exploitation validation
* Security findings documentation

---

# 📄 Full Report

A detailed penetration testing report, including the complete methodology and supporting screenshots, is available in the PDF included in this directory.

**[📥 View / Download the Complete Simple CTF Report](./simple%20CTF%20%281%29.pdf)**

---

# 📝 Conclusion

The Simple CTF lab demonstrated how several individual security weaknesses can be chained together to achieve complete system compromise.

The attack path involved:

**Exposed FTP → Information Disclosure → Weak Credentials → SSH Access → Excessive Sudo Privileges → Root Access**

The exercise provided practical experience with reconnaissance, service enumeration, credential auditing, Linux privilege escalation, and penetration-test reporting.

---

## ⚠️ Responsible Use

All techniques documented in this report were performed against an authorized TryHackMe training target.

These techniques should only be used in environments where explicit permission has been provided.

---

**Platform:** TryHackMe
**Room:** Simple CTF
**Report Type:** Educational Penetration Testing Lab
**Author:** Sunil
