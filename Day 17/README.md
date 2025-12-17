# 🎄 TryHackMe Advent of Cyber 2025 - Day 17 🎄

## CyberChef - Hoperation Save McSkidy: Multi-Lock Encoding Challenge

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/room/adventofcyber2025)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-45%20minutes-blue?style=for-the-badge)](https://tryhackme.com)
[![Day](https://img.shields.io/badge/Day-17-orange?style=for-the-badge)](https://tryhackme.com)

---

## 📖 Story Overview

McSkidy is imprisoned in King Malhare's Quantum Warren. Sir BreachBlocker III has implemented multiple security locks to prevent any escape. However, McSkidy managed to send vital clues using harmless bunny pictures.

One message revealed: **Five locks must be disabled** to secure an escape route.

The locks can be broken by:
- 🔍 Examining their logic
- 💬 Leveraging the guard chat system
- 🔐 Speaking their language (encoded communication)

**Your Mission**: Mount "Hoperation Save McSkidy" by breaking through five increasingly complex security locks using CyberChef, web inspection, and encoding/decoding techniques! 🐰🔓

---

## 🎯 Learning Objectives

By completing this challenge, you'll master:

- ✅ **Encoding vs Encryption** - Understanding fundamental differences
- ✅ **CyberChef Operations** - The Swiss Army knife of data transformation
- ✅ **Base64 Encoding** - Single and double encoding/decoding
- ✅ **XOR Operations** - Reversible cryptographic operations
- ✅ **Hash Cracking** - Using precomputed databases (CrackStation)
- ✅ **ROT Ciphers** - ROT13 and ROT47 transformations
- ✅ **HTTP Header Analysis** - Extracting hidden information
- ✅ **Web Developer Tools** - Network, Debugger, and Console tabs
- ✅ **Chained Operations** - Combining multiple transformations

---

## 📚 Core Concepts

### Encoding vs Encryption

| Aspect | Encoding | Encryption |
|--------|----------|------------|
| **Purpose** | Compatibility & Usability | Security & Confidentiality |
| **Process** | Standardized (no secret) | Algorithm + Secret Key |
| **Reversible** | Yes (easily) | Yes (with correct key) |
| **Security** | ❌ NO | ✅ YES |
| **Speed** | ⚡ Fast | 🐢 Slower |
| **Examples** | Base64, URL encoding, Hex | TLS, AES, RSA |

**Key Insight**: Encoding ensures compatibility. Encryption provides security. Never confuse the two!

---

### CyberChef: The Cyber Swiss Army Knife

**Interface Layout:**

```
┌────────────────────────────────────────────────────┐
│  OPERATIONS  │  RECIPE  │   INPUT   │   OUTPUT    │
├──────────────┼──────────┼───────────┼─────────────┤
│              │          │           │             │
│  • To Base64 │  Step 1  │  Your     │  Encoded    │
│  • From Base │  Step 2  │  data     │  or         │
│  • XOR       │  Step 3  │  goes     │  decoded    │
│  • ROT13     │  ...     │  here     │  result     │
│  • MD5       │          │           │             │
│  • [300+]    │          │           │             │
│              │          │           │             │
└────────────────────────────────────────────────────┘
```

**Features:**
- 🔧 300+ operations available
- 🔗 Chain multiple operations
- 💾 Save and share recipes
- ⚡ Real-time processing
- 🎯 Toggle operations on/off

---

### Web Developer Tools

**Access Developer Tools:**

| Browser | Shortcut | Menu Path |
|---------|----------|-----------|
| All | `F12` | Various paths |
| All | `Ctrl+Shift+I` (Win/Linux) | - |
| All | `Cmd+Option+I` (Mac) | - |

**Key Tabs for This Challenge:**

| Tab | Purpose | What to Look For |
|-----|---------|------------------|
| **Network** | HTTP requests/responses | Headers, response data, status codes |
| **Debugger** | JavaScript source code | Login logic, comments, encoding methods |
| **Console** | JavaScript execution | Errors, log messages, variables |

---

## 🎯 Challenge Overview

**Target**: `http://MACHINE_IP:8080`

### Universal Strategy (All Locks)

**Every lock follows this pattern:**

```
1. Identify Guard Name
   ├─ Check headers or page content
   └─ Encode to Base64 (username)

2. Find Additional Info
   ├─ Magic question (Locks 1-2)
   ├─ XOR key (Locks 3, 5)
   └─ Recipe ID (Lock 5)

3. Communicate with Guard
   ├─ Encode message to Base64
   ├─ Send via chat
   └─ Wait for encoded response (2-3 min from Lock 3+)

4. Analyze Login Logic
   ├─ Open Debugger tab
   ├─ Find encoding method
   └─ Note any comments or hints

5. Build Reverse Recipe
   ├─ Open CyberChef
   ├─ Chain operations in reverse order
   └─ Decode password

6. Login
   ├─ Username: Encoded guard name
   └─ Password: Decoded password
```

---

## 🔓 Lock 1: Outer Gate

### Challenge: Single Base64 Encoding

**Encoding Method**: Base64

**Investigation:**

| Step | Action | Tool | Finding |
|------|--------|------|---------|
| 1 | Find guard name | Network tab → Headers | Guard name |
| 2 | Encode to Base64 | CyberChef | Username ready |
| 3 | Find magic question | Headers | Question text |
| 4 | Encode question | CyberChef | Encoded question |
| 5 | Ask guard | Chat | Encoded password |
| 6 | Check login logic | Debugger tab | "Base64 encoded" |
| 7 | Decode password | CyberChef: From Base64 | Plaintext password |

### CyberChef Recipe (Decode)

```
Input: [Encoded password from guard]
────────────────────────────
Operation: From Base64
────────────────────────────
Output: Iamsofluffy
```

### Solution

**Password**: `Iamsofluffy`

**Key Learning**: Basic Base64 encoding/decoding fundamentals.

---

## 🔓 Lock 2: Outer Wall

### Challenge: Double Base64 Encoding

**Encoding Method**: Base64(Base64(password))

**New Complexity**: Encoding applied **twice**!

**Investigation:**

Login Logic shows: `"Password is encoded to Base64 twice"`

**This means**: Must decode Base64 **twice** to get plaintext!

### CyberChef Recipe (Decode)

```
Input: [Double-encoded password]
────────────────────────────
Operation 1: From Base64
────────────────────────────
Intermediate: [Still encoded]
────────────────────────────
Operation 2: From Base64
────────────────────────────
Output: Itoldyoutochangeit!
```

**Chain Structure**: From Base64 → From Base64

### Solution

**Password**: `Itoldyoutochangeit!`

**Key Learning**: Chaining operations—decode in reverse order of encoding.

---

## 🔓 Lock 3: Guard House

### Challenge: XOR + Base64 Encoding

**Encoding Method**: XOR(password, key) → Base64

**New Elements**:
- 🔑 XOR key required (from headers)
- ⏱️ Guards respond slowly (2-3 minutes)
- ❌ No more magic questions

**Important**: Keep chat messages short! Try: `"Password please."`

### XOR Theory

**XOR (Exclusive OR)** is a bitwise operation with a special property:

```
Original Data XOR Key = Encrypted
Encrypted XOR Key = Original Data
```

**Magic Property**: XORing twice with the same key returns the original!

**Test in CyberChef:**
```
Input: Hello
XOR(secret) → [encrypted]
XOR(secret) → Hello (back to original!)
```

### Investigation

| Item | Location | Example Value |
|------|----------|---------------|
| Guard Name | Headers/Page | GuardianGamma |
| XOR Key | Headers | `secret123` |
| Encoded Password | Guard chat | [Base64 string] |

### CyberChef Recipe (Decode)

```
Input: [Encoded password from guard]
────────────────────────────
Operation 1: From Base64
────────────────────────────
Intermediate: [XOR encrypted text]
────────────────────────────
Operation 2: XOR
  Key: secret123
  Key Format: UTF-8
────────────────────────────
Output: BugsBunny
```

### Solution

**Password**: `BugsBunny`

**Key Learning**: XOR is reversible—same operation encrypts and decrypts!

---

## 🔓 Lock 4: Inner Castle

### Challenge: MD5 Hash Cracking

**Encoding Method**: MD5(password)

**New Complexity**: One-way hash function!

**Important**: This is **not encoding**—it's **hashing**!

### MD5 Theory

**MD5 (Message-Digest Algorithm 5)**:
- Produces 128-bit (32 hexadecimal character) hash
- Designed to be **one-way** (cannot reverse mathematically)
- **BUT**: Precomputed databases exist for common passwords!

**Example**:
```
Input: password
MD5 Hash: 5f4dcc3b5aa765d61d8327deb882cf99
```

### Investigation

**Login Logic**: `if (MD5(input) === stored_hash)`

**Guard Response** (decoded): `2ac9cb7dc02b3c0083eb70898e549b63`

**This is an MD5 hash!** Cannot reverse—must crack.

### Hash Cracking with CrackStation

**Tool**: https://crackstation.net/

**Process:**
1. Decode guard's Base64 response
2. Copy MD5 hash
3. Paste into CrackStation
4. Click "Crack Hashes"
5. Retrieve plaintext password

**How It Works**: CrackStation has precomputed hashes for billions of common passwords.

### Solution

**Hash**: `2ac9cb7dc02b3c0083eb70898e549b63`  
**Cracked Password**: `passw0rd1`

**Key Learning**: 
- MD5 is broken for passwords
- Common passwords easily cracked
- Use bcrypt/Argon2 instead!

---

## 🔓 Lock 5: Prison Tower (FINAL)

### Challenge: Dynamic Encoding (Recipe-Based)

**New Mechanism**: Encoding method **changes** based on Recipe ID!

**McSkidy's Warning**:
*"Sir BreachBlocker III implemented different mechanisms for the last lock, which change occasionally. Match the correct approach when decoding."*

### Recipe ID System

**Recipe IDs** in headers determine the encoding method.

| Recipe ID | Forward Encoding | Reverse Decoding |
|-----------|------------------|------------------|
| **1** | ROT13 → Reverse → To Base64 | From Base64 → Reverse → ROT13 |
| **2** | Reverse → To Hex → To Base64 | From Base64 → From Hex → Reverse |
| **3** | XOR(key) → To Base64 → ROT13 | ROT13 → From Base64 → XOR(key) |
| **4** | ROT47 → To Base64 → ROT13 | ROT13 → From Base64 → ROT47 |

**Critical**: Check headers for `Recipe-ID` or similar field!

### Investigation

**Example: Recipe ID 3**

| Item | Location | Value |
|------|----------|-------|
| Recipe ID | Headers | `3` |
| XOR Key | Headers | `fortress` |
| Guard Name | Headers/Page | [Encode to Base64] |
| Encoded Password | Guard chat | [Wait 2-3 min] |

### CyberChef Recipe (Recipe ID 3)

**Forward Encoding** (what was done):
```
Password → XOR(fortress) → To Base64 → ROT13
```

**Reverse Decoding** (what we do):
```
Encoded → ROT13 → From Base64 → XOR(fortress) → Password
```

**CyberChef Recipe:**
```
Input: FgoeFrnpuOybpxre=
────────────────────────────
Operation 1: ROT13
  Amount: 13
────────────────────────────
Output 1: StuRSeapByObpxre=
────────────────────────────
Operation 2: From Base64
────────────────────────────
Output 2: [Binary data]
────────────────────────────
Operation 3: XOR
  Key: fortress
  Key Format: UTF-8
────────────────────────────
Output 3: 51rBr34chBl0ck3r
```

### Solutions (All Recipe IDs)

**Recipe ID 1**: From Base64 → Reverse → ROT13  
**Recipe ID 2**: From Base64 → From Hex → Reverse  
**Recipe ID 3**: ROT13 → From Base64 → XOR(key)  
**Recipe ID 4**: ROT13 → From Base64 → ROT47  

### Final Answers

**Password**: `51rBr34chBl0ck3r`  
**Flag**: `THM{M3D13V4L_D3C0D3R_4D3P7}`

**Key Learning**: Adaptive defenses require flexible thinking and tool mastery!

---

## 📊 Complete Solutions Summary

| Lock | Encoding Method | Password | Key Technique |
|------|----------------|----------|---------------|
| **1: Outer Gate** | Base64 (x1) | `Iamsofluffy` | Basic decoding |
| **2: Outer Wall** | Base64 (x2) | `Itoldyoutochangeit!` | Chained operations |
| **3: Guard House** | XOR + Base64 | `BugsBunny` | Reversible crypto |
| **4: Inner Castle** | MD5 Hash | `passw0rd1` | Hash cracking |
| **5: Prison Tower** | Dynamic (Recipe ID) | `51rBr34chBl0ck3r` | Adaptive approach |

**Final Flag**: `THM{M3D13V4L_D3C0D3R_4D3P7}`

---

## 🔧 CyberChef Operations Reference

### Essential Operations

| Operation | Purpose | Input → Output |
|-----------|---------|----------------|
| **To Base64** | Encode to Base64 | `Hello` → `SGVsbG8=` |
| **From Base64** | Decode from Base64 | `SGVsbG8=` → `Hello` |
| **XOR** | Bitwise XOR encryption | `Hello` + `key` → `[encrypted]` |
| **ROT13** | Caesar cipher (shift 13) | `Hello` → `Uryyb` |
| **ROT47** | Extended ROT for ASCII | `Hello` → `w6==@` |
| **MD5** | Hash to MD5 | `password` → `5f4dcc3b...` |
| **Reverse** | Reverse string | `Hello` → `olleH` |
| **To Hex** | Convert to hexadecimal | `Hello` → `48656c6c6f` |
| **From Hex** | Decode from hexadecimal | `48656c6c6f` → `Hello` |

### Recipe Examples

**Double Base64 Decode:**
```
From Base64 → From Base64
```

**XOR + Base64 Decode:**
```
From Base64 → XOR
  Key: secret123
```

**Recipe ID 3 (Lock 5):**
```
ROT13 → From Base64 → XOR
  Key: fortress
```

**Recipe ID 1:**
```
From Base64 → Reverse → ROT13
```

---

## 🎓 Key Takeaways

1. **🔐 Encoding ≠ Encryption ≠ Hashing**
   - **Encoding**: Compatibility (Base64, Hex)
   - **Encryption**: Security with keys (XOR, AES)
   - **Hashing**: One-way fingerprint (MD5, SHA)

2. **🔪 CyberChef is Essential**
   - 300+ operations available
   - Chain operations for complex tasks
   - Real-time processing and feedback
   - Save recipes for reuse

3. **🔗 Reverse Engineering Encoding**
   - Always work **backwards**
   - If encoded: A → B → C
   - Then decode: C → B → A

4. **🔑 XOR is Special**
   - Same operation encrypts and decrypts
   - XOR(XOR(data, key), key) = data
   - Requires key to reverse

5. **🔨 Hashes Can Be Cracked**
   - MD5/SHA1 broken for passwords
   - Precomputed databases (rainbow tables)
   - Use bcrypt, Argon2, or PBKDF2 instead

6. **🌐 Web Inspection Reveals All**
   - Network tab: Headers, responses
   - Debugger tab: Source code, logic
   - Console tab: Errors, variables

7. **⏱️ Patience with Communication**
   - Guards slow to respond (2-3 min)
   - Keep messages concise
   - Plan next steps while waiting

8. **🎯 Adaptive Defenses Work**
   - Recipe IDs force flexible thinking
   - Can't use same approach every time
   - Requires tool mastery and adaptability

---

## 🛡️ Defense Lessons

### What Defenders Should Learn

**1. Encoding Provides NO Security**
```
❌ NEVER rely on encoding (Base64, Hex) for protection
✅ Use proper encryption with strong keys (AES-256)
```

**2. MD5 is Completely Broken**
```
❌ NEVER use MD5 for password hashing
❌ NEVER use SHA1 for password hashing
✅ Use bcrypt (cost factor 12+)
✅ Use Argon2id
✅ Use PBKDF2 (100,000+ iterations)
```

**3. Security Through Obscurity Fails**
```
❌ Hidden encoding methods eventually discovered
✅ Use established cryptographic standards
✅ Assume attackers know your methods
```

**4. Dynamic Defenses Add Value**
```
✅ Changing methods increases attack complexity
✅ Forces attackers to adapt and spend time
⚠️ Must be implemented correctly
⚠️ Don't create false sense of security
```

**5. Defense in Depth Works**
```
✅ Multiple different security layers
✅ Each lock used different technique
✅ Significantly slows down attackers
✅ Provides detection opportunities
```

---

## 🔗 Related TryHackMe Rooms

Continue your crypto journey:

| Room | Focus Area | Difficulty | Duration |
|------|-----------|-----------|----------|
| **Encryption - Crypto 101** | Cryptography fundamentals | Easy | 60 min |
| **Hashing - Crypto 101** | Hash functions and attacks | Easy | 45 min |
| **John The Ripper** | Advanced password cracking | Easy | 60 min |
| **Crack the Hash** | Hash identification & cracking | Easy | 45 min |
| **Web Application Basics** | HTTP, headers, requests | Easy | 90 min |

---

## 📚 Additional Resources

### 🛠️ Tools & Utilities

| Tool | Purpose | Link |
|------|---------|------|
| **CyberChef** | Data transformation | [gchq.github.io/CyberChef](https://gchq.github.io/CyberChef/) |
| **CrackStation** | Online hash cracking | [crackstation.net](https://crackstation.net/) |
| **HashCat** | Advanced GPU cracking | [hashcat.net](https://hashcat.net/) |
| **John the Ripper** | Password cracking suite | [openwall.com](https://www.openwall.com/john/) |
| **Base64 Decode** | Quick Base64 operations | [base64decode.org](https://www.base64decode.org/) |

### 📖 Documentation
- [CyberChef Recipes Repository](https://github.com/mattnotmax/cyberchef-recipes) - Community recipes
- [Encoding vs Encryption Explained](https://danielmiessler.com/study/encoding-encryption-hashing-obfuscation/) - Clear guide
- [XOR Cipher](https://en.wikipedia.org/wiki/XOR_cipher) - Wikipedia
- [ROT13](https://en.wikipedia.org/wiki/ROT13) - Caesar cipher variant

### 📚 Books & Courses
- **"Serious Cryptography" by Jean-Philippe Aumasson** - Modern crypto
- **"Cryptography Engineering" by Ferguson, Schneier, Kohno** - Practical approach
- **"The Code Book" by Simon Singh** - Historical perspective

---


## 🤝 Contributing

Found additional recipes or techniques? Contributions welcome!

1. Fork the repository
2. Create feature branch (`git checkout -b feature/new-cyberchef-recipe`)
3. Commit changes (`git commit -am 'Add XYZ decoding recipe'`)
4. Push to branch (`git push origin feature/new-cyberchef-recipe`)
5. Open Pull Request

---

## 📜 License

This walkthrough is provided for educational purposes as part of TryHackMe's Advent of Cyber 2025 challenge.

**Disclaimer**: Encoding/decoding techniques should only be used for authorized security testing and CTF challenges.

---

## 🌟 Support

If this walkthrough helped you:
- ⭐ Star this repository

---

<div align="center">

### 🎄 Master the Recipes, Decode the World! 🎄

**Transform. Decode. Conquer.**

Made with ❤️ for the cybersecurity community

![TryHackMe](https://tryhackme.com/img/logo/tryhackme_logo_full.svg)

</div>

---

## Tags

`#TryHackMe` `#AdventOfCyber2025` `#CyberChef` `#Encoding` `#Decoding` `#Base64` `#XOR` `#MD5` `#HashCracking` `#ROT13` `#Cryptography` `#WebInspection` `#CTF`
