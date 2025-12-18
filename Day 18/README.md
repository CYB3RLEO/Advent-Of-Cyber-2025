# 🎄 TryHackMe Advent of Cyber 2025 - Day 18: Obfuscation - The Egg Shell File

![TryHackMe](https://img.shields.io/badge/TryHackMe-AoC%202025-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)
![Duration](https://img.shields.io/badge/Duration-30%20Minutes-blue?style=for-the-badge)
![Room](https://img.shields.io/badge/Topic-Obfuscation-orange?style=for-the-badge)

---

## 🎯 Overview

**Room**: Advent of Cyber 2025 - Day 18  
**Title**: Obfuscation - The Egg Shell File  
**Topic**: Obfuscation & Deobfuscation Techniques  
**Difficulty**: Easy  
**Duration**: 30 minutes

This challenge teaches essential obfuscation and deobfuscation techniques used by both attackers and defenders in cybersecurity. Learn to identify and reverse common obfuscation methods using CyberChef and hands-on PowerShell analysis.

---

## 📖 The Story

WareVille is in chaos following the appearance of a mysterious wormhole. Systems are malfunctioning, dashboards are spiking, and SOC alerts are firing continuously. 

McSkidy, the vigilant SOC analyst, notices a suspicious email claiming to be from `northpole-hr`—decorated with carrot emojis. The problem? There is no North Pole HR department. TBFC's HR operates from the South Pole.

Investigating further, McSkidy discovers a PowerShell file was downloaded from this phishing email. The script is filled with gibberish—random strings of characters that make no sense at first glance. This is **obfuscation**, a technique attackers use to hide malicious code and evade detection.

Your mission: Help McSkidy deobfuscate the suspicious PowerShell script and uncover the attacker's plans.

---

## 🎓 Learning Objectives

By completing this challenge, you will:

- ✅ Understand what obfuscation is and why attackers use it
- ✅ Learn the differences between encoding, encryption, and obfuscation
- ✅ Master common obfuscation techniques (ROT1, ROT13, Base64, XOR)
- ✅ Use CyberChef effectively for encoding and decoding
- ✅ Recognize patterns in obfuscated data
- ✅ Reverse layered obfuscation techniques
- ✅ Analyze obfuscated PowerShell scripts safely

---

## 🔌 Connection Details

### Target Machine Information

| Parameter | Value |
|-----------|-------|
| **IP Address** | `10.67.134.185` |
| **Username** | `Administrator` |
| **Password** | `@TryObfusc4t3M393!` |
| **Connection** | RDP or In-Browser Split View |

### Starting the Machine

1. Click **Start Machine** button in the TryHackMe room
2. Wait ~2 minutes for the machine to load
3. Use in-browser split view or connect via RDP

---

## 🔐 Obfuscation Techniques

### What is Obfuscation?

**Obfuscation** is the practice of making data intentionally difficult to read and analyze. It transforms readable information into gibberish while preserving its functionality.

#### Key Differences

| Technique | Purpose | Reversibility | Security |
|-----------|---------|---------------|----------|
| **Encoding** | Data compatibility | Easy (no key) | None |
| **Encryption** | Confidentiality | Requires key | High |
| **Obfuscation** | Obscurity | Pattern-based | Low-Medium |

---

### 1️⃣ ROT1 (Caesar Cipher - Shift 1)

Shifts each letter forward by 1 position in the alphabet.

**Example**:
```
Original: carrot coins go brr
ROT1:     dbsspu dpjot hp css
```

**Detection Pattern**:
- Common words look "one letter off"
- Spaces remain unchanged
- Easy to spot and reverse

---

### 2️⃣ ROT13 (Caesar Cipher - Shift 13)

Shifts characters forward by 13 positions (symmetric).

**Example**:
```
Original: the and
ROT13:    gur naq
```

**Detection Pattern**:
- Three-letter words look familiar but transformed
- `the` → `gur`, `and` → `naq`
- Spaces stay the same

---

### 3️⃣ Base64 Encoding

Encodes binary data into ASCII text using 64 characters.

**Example**:
```
Original: carrot
Base64:   Y2Fycm90
```

**Detection Pattern**:
- Long alphanumeric strings
- Contains `A-Z`, `a-z`, `0-9`, `+`, `/`
- Often ends with `=` or `==` (padding)

---

### 4️⃣ XOR (Exclusive OR)

Combines each byte with a key using XOR operation.

**Example**:
```
Original: carrot supremacy
Key:      a (hex)
XOR:      ikxxe~*yzxogkis!
```

**Detection Pattern**:
- Random symbols and characters
- Same length as original
- Short key reuse may show repetitive patterns

---

### 🧅 Layered Obfuscation

Attackers often combine multiple techniques:

**Example Process**:
1. Original script
2. → Gzip compression
3. → XOR with key 'x'
4. → Base64 encoding

**Result**: `H4sIADKZ42gA/32PT2sqQRDE7/Mp...`

**Reversal Process** (opposite order):
1. From Base64
2. → XOR with key 'x'
3. → Gunzip

---

## 🚀 Complete Walkthrough

### Part 1: Deobfuscating the C2 URL

#### Step 1: Open the PowerShell Script

1. Double-click `SantaStealer.ps1` on the Desktop
2. Visual Studio Code will open (takes a moment)
3. Navigate to the "Start here" section

#### Step 2: Follow Script Instructions

The script contains comments that guide you through deobfuscating a Command and Control (C2) server URL. Read carefully and make the necessary changes.

**Key observations**:
- Look for Base64 encoded strings
- Identify the obfuscation pattern
- Use the clues in the comments to decode properly

#### Step 3: Save and Run the Script

Save the file (`Ctrl+S`), then open PowerShell:

**Windows Key** → type "PowerShell" → **Enter**

```powershell
# Navigate to Desktop
PS C:\Users\Administrator> cd .\Desktop\

# Execute the script
PS C:\Users\Administrator\Desktop> .\SantaStealer.ps1
```

#### 🏁 First Flag

After running the script, you'll receive:

```
THM{C2_De0bfuscation_29838}
```

---

### Part 2: Obfuscating the API Key

Now you'll practice **obfuscating** data, simulating how attackers hide sensitive information.

#### Step 1: Find Part 2 Instructions

Scroll to the "Part 2" section in the script comments.

#### Step 2: Use CyberChef for XOR Obfuscation

1. Open [CyberChef](https://gchq.github.io/CyberChef/)
2. Paste the API key in the **Input** box
3. Drag **XOR** operation to **Recipe**
4. Configure settings:
   - **Key**: `a`
   - **Mode**: `HEX`
5. Click **BAKE!**
6. Copy the obfuscated output

#### Step 3: Update and Run Script

1. Paste the obfuscated API key into the script
2. Save the file (`Ctrl+S`)
3. Run the script again:

```powershell
PS C:\Users\Administrator\Desktop> .\SantaStealer.ps1
```

#### 🏁 Second Flag

After running the updated script:

```
THM{API_Obfusc4tion_ftw_0283}
```

---

## 🎯 Flags & Answers

| Question | Answer |
|----------|--------|
| What is the first flag you get after deobfuscating the C2 URL and running the script? | `THM{C2_De0bfuscation_29838}` |
| What is the second flag you get after obfuscating the API key and running the script again? | `THM{API_Obfusc4tion_ftw_0283}` |
| If you want to learn more about Obfuscation, check out our Obfuscation Principles room! | No answer needed |

---

## 📚 Command Reference

### PowerShell Commands

| Command | Description |
|---------|-------------|
| `cd .\Desktop\` | Navigate to Desktop directory |
| `.\SantaStealer.ps1` | Execute PowerShell script |
| `Get-Content .\file.txt` | Read file contents |
| `Set-Content .\file.txt "data"` | Write to file |

### CyberChef Operations

| Operation | Purpose | Common Use |
|-----------|---------|------------|
| **From Base64** | Decode Base64 | Reverse Base64 encoding |
| **To Base64** | Encode to Base64 | Create Base64 strings |
| **XOR** | XOR encryption | Obfuscate/deobfuscate with key |
| **ROT13** | Caesar cipher (13) | Decode/encode ROT13 |
| **Magic** | Auto-detect encoding | Unknown obfuscation type |
| **Gunzip** | Decompress gzip | Extract compressed data |

### Pattern Recognition Quick Guide

```
Base64:     Y2Fycm90c3VwcmVtYWN5
ROT13:      pneebg fhcerzpnl
ROT1:       dbsspu tvqsfnbdz
XOR:        ikxxe~*yzxogkis!
```

---

## 🧠 Key Concepts

### Why Attackers Use Obfuscation

1. **Evade Signature Detection**: Security tools can't detect malicious keywords if they're obfuscated
2. **Delay Analysis**: Takes time for analysts to decode and understand
3. **Hide Infrastructure**: C2 servers, API keys, and domains are concealed
4. **Bypass Filters**: Email and web filters may not recognize obfuscated content

### Defense Strategies

#### 1. Behavioral Analysis
Focus on what code **does**, not what it **looks like**.

#### 2. Sandboxing
Execute suspicious scripts in isolated environments to observe behavior safely.

#### 3. Deobfuscation Tools
- **CyberChef**: Web-based encoding/decoding
- **de4dot**: .NET deobfuscator
- **PowerShell AST**: Parse PowerShell syntax trees

#### 4. Pattern Recognition
Train analysts to spot common obfuscation signatures:
- Base64 padding (`=`, `==`)
- High entropy strings
- Unusual character distributions

#### 5. Logging and Monitoring
- Enable PowerShell script block logging
- Monitor process creation events
- Track network connections from scripts

---

## 💡 Key Takeaways

✅ **Obfuscation** transforms readable data into gibberish to evade detection

✅ **Common techniques** include ROT ciphers, Base64, and XOR encryption

✅ **CyberChef** is an essential tool for security professionals

✅ **Pattern recognition** helps identify obfuscation methods quickly

✅ **Layered obfuscation** requires systematic reverse-order deobfuscation

✅ **PowerShell** is frequently targeted due to its power and prevalence

✅ **Defense** requires both technical tools and analytical thinking

---

## 🔗 Related Rooms

Continue your learning with these TryHackMe rooms:

- **Obfuscation Principles**: Deep dive into obfuscation techniques
- **PowerShell for Pentesters**: Offensive PowerShell skills
- **Malware Analysis**: Analyze obfuscated malware samples
- **Intro to Detection Engineering**: Build detection rules

---

## 🤝 Contributing

Found an issue or want to improve this walkthrough?

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

---

## 📄 License

This walkthrough is created for educational purposes as part of TryHackMe's Advent of Cyber 2025.

---

## 🙏 Acknowledgments

- **TryHackMe** for creating Advent of Cyber 2025
- **GCHQ** for developing CyberChef
- The cybersecurity community for sharing knowledge

---

## 📬 Contact

Questions or feedback? Reach out:

- **TryHackMe**: [Profile](https://tryhackme.com)
- **GitHub**: [Issues](https://github.com/yourusername/aoc2025-day18/issues)

---

**Happy Hacking! 🎄🔐**

*Stay curious, stay vigilant, and keep learning!*
