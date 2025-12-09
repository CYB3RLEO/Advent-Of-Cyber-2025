# 🎄 Advent of Cyber 2025 - Day 9: Passwords - A Cracking Christmas

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-30%20min-blue?style=for-the-badge)](https://tryhackme.com)
[![Passwords](https://img.shields.io/badge/Topic-Password%20Cracking-critical?style=for-the-badge)](https://tryhackme.com)

## 📖 Overview

Day 9 explores password cracking techniques for encrypted files. Learn how attackers recover weak passwords from PDFs and ZIP archives, and how defenders can detect and prevent these attacks.

### 🎯 Learning Objectives

- Understand password-based file encryption
- Master dictionary and brute-force attack techniques
- Learn to crack encrypted PDFs and ZIP files
- Recognize detection opportunities for cracking activity
- Implement strong password defenses

## 🔍 Investigation Summary

**Threat Actor:** Sir Carrotbane  
**Target:** TBFC encrypted archives ("North Pole Asset List")  
**Attack Type:** Offline password cracking  
**Tools Used:** pdfcrack, John the Ripper  
**Vulnerability:** Weak, predictable passwords  

## 🔐 Password Encryption Fundamentals

### How It Works

```
User Password → Key Derivation Function → Encryption Key → Encrypted Data
```

**Critical Point:** Security depends ENTIRELY on password strength.

### File Format Comparison

| Format | Encryption | KDF | Weakness |
|--------|-----------|-----|----------|
| **PDF** | RC4/AES | Varies | Legacy modes supported |
| **ZIP** | ZipCrypto/AES | PBKDF2 | ZipCrypto very weak |
| **7Z** | AES-256 | Strong | Usually secure |

### The Offline Attack Advantage

Once attacker has the file:
- ❌ No rate limiting
- ❌ No lockouts
- ❌ No alerts
- ✅ Unlimited attempts
- ✅ Can try billions of passwords

## 🎯 Attack Methodologies

### Dictionary Attacks

**Definition:** Test passwords from predefined wordlist

**Common Wordlists:**
```bash
# rockyou.txt (14+ million passwords from breaches)
/usr/share/wordlists/rockyou.txt

# SecLists
https://github.com/danielmiessler/SecLists
```

**Why They Work:**
- Users choose predictable passwords
- Patterns repeat across breaches
- Contextual passwords (company names, seasons)

**Speed:** 10,000 - 1,000,000+ passwords/second

### Mask Attacks

**Definition:** Brute force with constraints

**Mask Syntax:**
```
?l = lowercase letter
?u = uppercase letter
?d = digit
?s = special character
?a = all of above
```

**Examples:**
```
?l?l?l?d?d          # abc12
?u?l?l?l?l?d?d?d?d  # Winter2024
?d?d?d?d            # 1234 (PIN)
```

## 📝 Complete Walkthrough

### Setup

**SSH Access:**
```bash
ssh ubuntu@10.67.144.87
Password: AOC2025Ubuntu!
```

**Navigate to Files:**
```bash
cd Desktop
ls
# flag.pdf  flag.zip  john  mate-terminal.desktop
```

### Phase 1: File Identification

**Identify Types:**
```bash
file flag.pdf
# PDF document, version 1.7, 1 page(s)

file flag.zip
# Zip archive data, at least v2.0 to extract, compression method=AES Encrypted
```

### Phase 2: Cracking PDF Password 📄

**Tool:** pdfcrack

**Command:**
```bash
pdfcrack -f flag.pdf -w /usr/share/wordlists/rockyou.txt
```

**Output:**
```
PDF version 1.7
Security Handler: Standard
V: 2
R: 3
P: -1060
Length: 128
Encrypted Metadata: True
FileID: 3792b9a3671ef54bbfef57c6fe61ce5d
...
Average Speed: 29538.5 w/s
...
found user-password: 'naughtylist'
```

**Analysis:**
- Speed: ~30,000 passwords/second
- Time: Seconds
- Password: `naughtylist`

**Extract Flag:**
```bash
qpdf --decrypt --password=naughtylist flag.pdf decrypted.pdf
cat decrypted.pdf
```

**🚩 Answer 1:** `THM{Cr4ck1ng_PDFs_1s_34$y}`

### Phase 3: Cracking ZIP Password 📦

**Tool:** John the Ripper

**Step 1: Extract Hash**
```bash
zip2john flag.zip > ziphash.txt
```

**Step 2: Dictionary Attack**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt ziphash.txt
```

**Output:**
```
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Cost 1 (HMAC size [KiB]) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
winter4ever      (flag.zip/flag.txt)     
1g 0:00:00:00 DONE (2025-12-09 18:37) 2.222g/s 9102p/s 9102c/s 9102C/s friend..sahara
```

**Analysis:**
- Speed: 9,102 passwords/second (slower due to PBKDF2)
- Time: < 1 second
- Password: `winter4ever`

**Extract Flag:**
```bash
unzip flag.zip
# Enter password: winter4ever

cat flag.txt
```

**🚩 Answer 2:** `THM{Cr4ck1n6_z1p$_1s_34$yyyy}`

## 🏆 Challenge Answers

| Question | Answer |
|----------|--------|
| Flag in encrypted PDF | `THM{Cr4ck1ng_PDFs_1s_34$y}` |
| Flag in encrypted ZIP | `THM{Cr4ck1n6_z1p$_1s_34$yyyy}` |

## 🛠️ Tool Reference

### pdfcrack

**Purpose:** PDF password recovery

```bash
# Basic usage
pdfcrack -f file.pdf -w wordlist.txt

# With constraints
pdfcrack -f file.pdf -w wordlist.txt -n 8 -m 12
# -n: minimum length
# -m: maximum length
```

### John the Ripper

**Purpose:** Universal password cracker

**Workflow:**
```bash
# 1. Extract hash
zip2john file.zip > hash.txt
pdf2john file.pdf > hash.txt
rar2john file.rar > hash.txt

# 2. Crack
john --wordlist=rockyou.txt hash.txt

# 3. Show results
john --show hash.txt
```

**Common Flags:**
- `--wordlist=FILE`: Dictionary attack
- `--rules`: Apply word mangling
- `--incremental`: Brute force
- `--format=FORMAT`: Specify hash type

### Hashcat

**Purpose:** GPU-accelerated cracking

**Hash Modes:**
```bash
-m 13600  # WinZip
-m 10500  # PDF 1.4-1.6
-m 10700  # PDF 1.7 Level 8
-m 0      # MD5
-m 1000   # NTLM
```

**Attack Modes:**
```bash
-a 0  # Dictionary
-a 3  # Brute force/Mask
-a 6  # Hybrid
```

**Example:**
```bash
hashcat -m 13600 -a 0 hash.txt rockyou.txt
```

### fcrackzip

**Purpose:** ZIP password recovery

```bash
# Dictionary attack
fcrackzip -u -D -p rockyou.txt file.zip

# Brute force
fcrackzip -u -b -l 4-6 file.zip
# -u: use unzip to verify
# -D: dictionary mode
# -b: brute force mode
# -l: password length range
```

## 🔍 Detection & Defense

### Detection Indicators

#### Process Monitoring

**Windows (Sysmon):**
```xml
<RuleGroup name="PasswordCracking">
  <ProcessCreate onmatch="include">
    <Image condition="contains any">
      \john.exe;
      \hashcat.exe;
      \pdfcrack.exe;
      \fcrackzip.exe
    </Image>
  </ProcessCreate>
</RuleGroup>
```

**Linux (auditd):**
```bash
auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/john -k crack_exec
auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/hashcat -k crack_exec
```

#### Command-Line Indicators

**Look for:**
- `--wordlist`, `-w`, `--rules`
- `--mask`, `-a 3`, `-m`
- `rockyou.txt`, `SecLists`
- `zip2john`, `pdf2john`

#### GPU Utilization

**Linux:**
```bash
nvidia-smi
# Shows: process name, GPU%, memory
```

**Indicators:**
- 95-100% GPU utilization
- Sustained high usage
- Processes: hashcat, john
- On non-graphics workstation

#### File Access

**Monitor:**
```bash
# Wordlist access
auditctl -w /usr/share/wordlists/rockyou.txt -p r -k wordlists

# Potfile writes
auditctl -w /home/*/.john/john.pot -p w -k potfile
auditctl -w /home/*/.hashcat/hashcat.potfile -p w -k potfile
```

### Sigma Detection Rule

```yaml
title: Password Cracking Tools Execution
id: 9f2f4d3e-4c16-4b0a-bb3a-7b1c6c001234
status: experimental
logsource:
  product: windows
  category: process_creation
detection:
  selection_name:
    Image|endswith:
      - '\john.exe'
      - '\hashcat.exe'
      - '\fcrackzip.exe'
      - '\pdfcrack.exe'
  selection_cmd:
    CommandLine|contains:
      - '--wordlist'
      - 'rockyou.txt'
      - 'zip2john'
      - 'pdf2john'
      - '--mask'
  condition: selection_name or selection_cmd
level: medium
```

### Incident Response Playbook

**1. Immediate Actions:**
```
✓ Isolate host (if malicious)
✓ Capture process list
✓ Preserve working directory
✓ Collect memory dump
```

**2. Evidence Collection:**
```
✓ Shell history
✓ Potfiles (cracked passwords)
✓ Wordlists used
✓ Hash files
✓ Target encrypted files
✓ GPU logs (nvidia-smi output)
```

**3. Analysis:**
```
✓ Which files were cracked?
✓ What data was accessed?
✓ Was this authorized?
✓ Any lateral movement?
✓ Data exfiltration?
```

**4. Remediation:**
```
✓ Rotate compromised passwords
✓ Re-encrypt with strong passwords
✓ Implement MFA
✓ User education
✓ Monitor for tool deployment
```

## 🛡️ Defensive Measures

### Strong Password Policy

**Requirements:**
```
✓ 16+ characters minimum
✓ Truly random (use password manager)
✓ Unique per file/service
✓ No dictionary words
✓ No predictable patterns
```

**Strength Comparison:**
```
Weak: Password123!           # 12 chars, predictable
Better: correct-horse-battery-staple  # 28 chars, random words
Best: Ky9$mP2#qL5*xF3&       # 16 chars, truly random
```

### Password Strength Testing

**Estimate crack time:**
```python
# Lowercase only, 8 chars
26^8 = 208 billion combinations
At 1 billion/sec = 208 seconds

# Mixed case + numbers + symbols, 16 chars
95^16 = astronomical (centuries)
```

### Proactive Auditing

**Test your own files:**
```bash
# Test PDF
pdfcrack -f your-file.pdf -w rockyou.txt

# Test ZIP
zip2john your-file.zip > hash.txt
john --wordlist=rockyou.txt hash.txt
```

**If cracked in < 1 hour → Password is too weak**

### Password Manager Benefits

✅ Generates truly random passwords  
✅ Stores securely  
✅ Unique per file/service  
✅ Long passwords (20+ chars)  
✅ No human patterns  

## 📊 Attack Speed Comparison

| Hash Type | Speed (CPU) | Speed (GPU) |
|-----------|-------------|-------------|
| MD5 | 100M/s | 50B/s |
| SHA1 | 50M/s | 20B/s |
| PBKDF2-SHA1 (ZIP) | 10K/s | 1M/s |
| bcrypt | 1K/s | 10K/s |
| scrypt | 100/s | 1K/s |

**Key Point:** Slower is better for defenders. PBKDF2, bcrypt, scrypt slow down attacks.

## 🔗 Related Resources

### TryHackMe Rooms
- [Password Attacks](https://tryhackme.com) - Comprehensive course
- [John the Ripper](https://tryhackme.com) - Tool deep dive
- [Hash Cracking](https://tryhackme.com) - Advanced techniques

### External Resources
- [Hashcat Wiki](https://hashcat.net/wiki/)
- [John the Ripper Docs](https://www.openwall.com/john/doc/)
- [Have I Been Pwned](https://haveibeenpwned.com/) - Check if passwords leaked

### Wordlists
- [rockyou.txt](https://github.com/brannondorsey/naive-hashcat/releases/tag/data) - 14M passwords
- [SecLists](https://github.com/danielmiessler/SecLists) - Curated lists
- [CrackStation](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm) - 1.5B passwords

## 🎓 Key Takeaways

### Password Weakness Analysis

**Why `naughtylist` Failed:**
- ❌ Dictionary word
- ❌ Seasonal/predictable theme
- ❌ All lowercase
- ❌ No complexity
- ❌ In rockyou.txt

**Why `winter4ever` Failed:**
- ❌ Common word
- ❌ Predictable pattern (word + number)
- ❌ Seasonal theme
- ❌ Leetspeak substitution (4 = for)
- ❌ In rockyou.txt

### Critical Lessons

**For Everyone:**
✅ Length > Complexity (but both matter)  
✅ Use password managers  
✅ Unique passwords per file/service  
✅ Test your own security proactively  

**For Attackers/Red Team:**
✅ Dictionary attacks first (fast wins)  
✅ GPU acceleration for supported formats  
✅ Offline cracking is powerful  
✅ But leaves process traces  

**For Defenders/Blue Team:**
✅ Monitor for cracking tools  
✅ Watch GPU utilization  
✅ Enforce strong passwords  
✅ Educate users  
✅ Regular password audits  

## 🤝 Contributing

Improvements welcome! Open a PR.

## 📜 License

Educational purposes only. All credit to TryHackMe and Advent of Cyber 2025.

---

⭐ **Found this helpful? Star the repo!**

🔐 **Day 9/25 Complete - Strong passwords save Christmas!**
