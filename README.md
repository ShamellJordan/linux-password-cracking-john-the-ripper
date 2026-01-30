# linux-password-cracking-john-the-ripper
# John the Ripper – Linux Password Cracking (Yescrypt) Lab

## Overview

This lab demonstrates a real-world post-exploitation credential attack in which Linux password hashes are extracted from `/etc/shadow` and cracked offline using **John the Ripper Jumbo**. The lab shows how attackers recover plaintext passwords, how modern hashing algorithms work, what obstacles attackers encounter, and why weak passwords remain dangerous even when strong hashing is used.

The lab is written from both a red-team execution and blue-team / SOC defense perspective.

---

## Objectives

- Understand how Linux stores authentication credentials
- Extract password hashes from `/etc/shadow`
- Identify modern password hashing algorithms (yescrypt)
- Crack passwords using dictionary and targeted attacks
- Troubleshoot tooling limitations
- Demonstrate multi-user credential compromise
- Explain defensive and SOC implications

---

## Lab Environment

Operating System: Ubuntu Linux (VirtualBox VM)  
Tool: John the Ripper Jumbo (Snap installation)  
Hash Type: yescrypt (`$y$`)  
Wordlists: RockYou + custom targeted list  
Access Level: Root (simulated post-exploitation)

---

## Why This Lab Matters

Modern Linux systems use yescrypt, a strong, memory-hard hashing algorithm designed to resist offline cracking. However, strong hashing does not compensate for weak passwords.

If an attacker gains access to `/etc/shadow`:
- Cracking happens offline
- No logs are generated
- Detection is extremely difficult

This lab demonstrates why prevention (password hygiene, access control, MFA) is more effective than detection.

---

## Lab Setup

### Step 1: Create Victim Accounts

Two users were created to simulate multiple compromised accounts:

sudo adduser victim1  
sudo adduser victim2  

Weak passwords were intentionally used to demonstrate real-world failure cases.

---

### Step 2: Extract Password Hashes

Linux stores user data in `/etc/passwd` and password hashes in `/etc/shadow`. These files were combined using `unshadow`:

sudo unshadow /etc/passwd /etc/shadow > hashes.txt  

Verification:

grep victim hashes.txt  

Example output:

victim1:$y$j9T$...  
victim2:$y$j9T$...

---

## Hash Analysis

The `$y$` prefix indicates **yescrypt**, a modern memory-hard hashing algorithm. Despite this protection, weak passwords were successfully cracked.

---

## Tooling Challenge Encountered

The default Ubuntu repository installs John the Ripper 1.8 (core), which does not support yescrypt. Initial cracking attempts resulted in:

No password hashes loaded

To resolve this, John the Ripper Jumbo was installed via Snap:

sudo snap install john-the-ripper  

Verification:

/snap/bin/john  

This mirrors real attacker behavior: adapting tooling when encountering modern defenses.

---

## Password Cracking Methodology

### Dictionary Attack (RockYou)

The RockYou password list was downloaded manually:

wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt  

Attack command:

/snap/bin/john --format=crypt --users=victim1,victim2 --wordlist=rockyou.txt hashes.txt  

Result:

victim2:qwerty123  

---

### Targeted Custom Wordlist

Because one password was not present in RockYou, a targeted list was created:

quicklist.txt contents:

password  
password123  
Password123  
password123!  
password@123  
victim1  
Victim1  
admin123  
letmein  

Attack command:

/snap/bin/john --format=crypt --users=victim1 --wordlist=quicklist.txt hashes.txt  

---

## Final Results (Proof of Compromise)

Command:

/snap/bin/john --show --users=victim1,victim2 hashes.txt  

Example output:

victim1:password123  
victim2:qwerty123  

2 password hashes cracked, 0 left

Both user accounts were successfully compromised.

---

## Attack Summary

User creation: Completed  
Hash extraction: Completed  
Hash identification (yescrypt): Completed  
Tool upgrade (John Jumbo): Completed  
Dictionary attack: Successful  
Targeted attack: Successful  
Multi-user compromise: Successful  

---

## MITRE ATT&CK Mapping

T1003.008 – OS Credential Dumping: /etc/shadow  
T1110.002 – Password Cracking: Dictionary Attack  
T1110.003 – Password Cracking: Brute Force (attempted)  

---

## Defensive & SOC Takeaways

Why detection is difficult:
- Cracking occurs offline
- No authentication logs are generated
- No network traffic is produced

Effective mitigations:
- Strong password policies
- Long passphrases
- Multi-factor authentication
- Restrict access to `/etc/shadow`
- Monitor privilege escalation
- Limit sudo access
- Detect post-compromise lateral movement

---

## Skills Demonstrated

- Linux authentication internals
- Credential harvesting techniques
- Hash identification and analysis
- Offline password cracking
- Tool troubleshooting and adaptation
- Red-team methodology
- Blue-team defensive reasoning
- Professional technical documentation

---

## Disclaimer

This lab was conducted in a controlled environment for educational purposes only. No unauthorized systems were accessed.

---

## Author

Built as part of a hands-on cybersecurity learning path focused on SOC Analyst and Blue Team readiness.
