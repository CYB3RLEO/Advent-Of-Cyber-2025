# 🎄 TryHackMe Advent of Cyber 2025 - Day 13 🎄

## YARA Rules - YARA Mean One!: Master Pattern Matching for Threat Detection

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/room/adventofcyber2025)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-60%20minutes-blue?style=for-the-badge)](https://tryhackme.com)
[![Day](https://img.shields.io/badge/Day-13-orange?style=for-the-badge)](https://tryhackme.com)

---

## 📖 Story Overview

The chaos deepens at The Best Festival Company (TBFC). When McSkidy went missing, uncertainty gripped the organization. But even in her disappearance, McSkidy was working tirelessly to help the blue team.

Following the crisis communication process, McSkidy sent a collection of images from an anonymous location. These images appeared to be Easter preparations, but they contained something more—a hidden message encoded within.

According to the crisis protocol, messages can be embedded in image folders using specific keywords that can be decoded if you know what to look for. The blue team must create a YARA rule to scan the directory, extract code words following the pattern, and decode McSkidy's crucial message.

**Your Mission**: Wield the power of YARA to uncover McSkidy's hidden message and discover her location. The fate of SOC-mas depends on decoding this intelligence! 🔍🎄

---

## 🎯 Learning Objectives

By completing this challenge, you'll master:

- ✅ **YARA Fundamentals** - Understanding what YARA is and how it works
- ✅ **Use Cases** - When and why to deploy YARA rules in real scenarios
- ✅ **String Types** - Text strings, hexadecimal patterns, and regex
- ✅ **Rule Writing** - Creating effective detection rules from scratch
- ✅ **Practical Detection** - Using YARA to hunt for malicious indicators
- ✅ **Intelligence Extraction** - Decoding hidden messages in files

---

## 🔍 What is YARA?

**YARA** (Yet Another Recursive Acronym) is a powerful pattern-matching tool designed to identify and classify malware by searching for unique patterns—the digital fingerprints left behind by attackers.

Think of YARA as a detective's notebook for cyber defenders:
- 🔎 Instead of dusting for prints, YARA scans code, files, and memory
- 🎯 It searches for subtle traces that reveal a threat's identity
- 🛡️ Acts as a silent guardian, uncovering malicious activity others might miss

### Why YARA Matters

In the world of cyberattacks, defenders face an endless stream of:
- 🚨 Alerts flooding SOC dashboards
- 📁 Suspicious files with unknown origins
- 🌐 Anomalous network fragments hiding threats

Not every threat stands out—some hide in plain sight, disguised as harmless documents or scripts. **That's where YARA shines.**

YARA gives defenders the power to detect malware by its **behavior and patterns**, not just by name. You can:
- 📝 Define your own rules
- 🔄 Share rules with the community
- 🎨 Customize what constitutes "malicious" behavior
- ⚡ Detect variants before antivirus signatures exist

---

## 🛡️ When to Use YARA

Defenders rely on YARA in various critical situations:

### 1. 🔬 Post-Incident Analysis
Verify whether traces of malware found on one compromised host still exist elsewhere in the environment.

**Example**: After detecting ransomware on a workstation, scan the entire network to find all infected systems.

### 2. 🎯 Threat Hunting
Proactively search through systems and endpoints for signs of known or related malware families.

**Example**: Hunt for indicators of APT groups across your infrastructure before they strike.

### 3. 🌐 Intelligence-Based Scans
Apply shared YARA rules from security researchers to detect new indicators of compromise.

**Example**: Use community rules from GitHub to identify zero-day exploits.

### 4. 💾 Memory Analysis
Examine active processes in memory dumps for malicious code fragments.

**Example**: Analyze a memory dump from a suspicious process to identify injected shellcode.

---

## 💎 YARA's Core Values

| Value | Description | Impact |
|-------|-------------|--------|
| ⚡ **Speed** | Quickly scans large sets of files or systems | Process thousands of files in seconds |
| 🎨 **Flexibility** | Detects text strings, binary patterns, and complex logic | Adapt to any threat scenario |
| 🎛️ **Control** | Analysts define exactly what they consider malicious | Customize detection for your environment |
| 🤝 **Shareability** | Rules can be reused and improved by the community | Collective defense against threats |
| 👁️ **Visibility** | Connects scattered clues into a clear attack picture | Transform raw data into actionable intelligence |

**Bottom Line**: YARA empowers defenders to move from **passive monitoring** to **active hunting**, turning intelligence into action before attackers strike.

---

## 📝 Anatomy of a YARA Rule

Every YARA rule consists of **three key components**:

```yara
rule TBFC_KingMalhare_Trace
{
    meta:
        author = "Defender of SOC-mas"
        description = "Detects traces of King Malhare's malware"
        date = "2025-12-13"
    
    strings:
        $s1 = "rundll32.exe" fullword ascii
        $s2 = "msvcrt.dll" fullword wide
        $url1 = /http:\/\/.*malhare.*/ nocase
    
    condition:
        any of them
}
```

### 1. 📋 Metadata Section
Information about the rule itself:
- **Author**: Who created it
- **Description**: What it detects
- **Date**: When it was written
- **Additional fields**: Version, reference, confidence level

**Best Practice**: Always include comprehensive metadata! When your rule collection grows, clear metadata saves countless hours.

### 2. 🔤 Strings Section
The actual clues YARA searches for:
- Text strings (ASCII/Unicode)
- Hexadecimal byte patterns
- Regular expressions

### 3. ⚖️ Condition Section
The logic that determines when the rule triggers:
- Boolean operators (and, or, not)
- Comparisons (filesize, entrypoint)
- String counts and positions

---

## 🔤 Understanding Strings

Strings are the fingerprints that YARA searches for when scanning files, memory, or other data sources.

### 1. 📝 Text Strings

The simplest and most common type—words or text fragments.

```yara
rule TBFC_Simple_Text
{
    strings:
        $keyword = "Christmas"
    
    condition:
        $keyword
}
```

#### Text String Modifiers

Attackers hide code by changing text appearance. YARA counters these obfuscation methods:

| Modifier | Purpose | Example |
|----------|---------|---------|
| `nocase` | Case-insensitive matching | `$xmas = "Christmas" nocase` |
| `wide` | Match Unicode (2-byte) characters | `$xmas = "Christmas" wide` |
| `ascii` | Match ASCII (1-byte) characters | `$xmas = "Christmas" ascii` |
| `xor` | Detect XOR-encoded strings | `$hidden = "Malhare" xor` |
| `base64` | Detect Base64-encoded strings | `$b64 = "SOC-mas" base64` |
| `fullword` | Match only complete words | `$exe = "run.exe" fullword` |

**Combining Modifiers**:
```yara
strings:
    $multi = "Christmas" wide ascii nocase
```

### 2. 🔢 Hexadecimal Strings

Malicious code often hides in raw bytes. Hex strings detect specific byte patterns.

```yara
rule TBFC_Hex_Detection
{
    strings:
        $mz = { 4D 5A }                    // MZ header (PE file)
        $pattern = { E3 41 ?? C8 }         // ?? = wildcard byte
        $range = { 48 8B [2-4] 48 89 }     // [2-4] = 2-4 wildcard bytes
    
    condition:
        $mz at 0 and any of ($pattern, $range)
}
```

**Wildcard Usage**:
- `??` - Any single byte
- `[2-4]` - 2 to 4 bytes
- `[0-10]` - 0 to 10 bytes

### 3. 🎯 Regular Expression Strings

Flexible patterns for detecting variants with structural similarities.

```yara
rule TBFC_Regex_Detection
{
    strings:
        $url = /http:\/\/.*malhare.*/ nocase
        $cmd = /powershell.*-enc\s+[A-Za-z0-9+\/=]+/ nocase
        $ip = /\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}/
    
    condition:
        any of them
}
```

⚠️ **Performance Tip**: Regex is powerful but can be slow. Use specific patterns and avoid overly broad expressions like `.*`.

---

## ⚖️ Understanding Conditions

The condition section is the **decision engine** of your YARA rule.

### Basic Conditions

| Condition | Description | Use Case |
|-----------|-------------|----------|
| `$string` | Match single string | Simple detection |
| `any of them` | Match any defined string | Broad detection |
| `all of them` | Match all strings | High confidence detection |
| `2 of them` | Match at least 2 strings | Balanced approach |

### Logical Operators

```yara
condition:
    ($s1 or $s2) and not $benign
```

| Operator | Purpose |
|----------|---------|
| `and` | Both conditions must be true |
| `or` | At least one condition must be true |
| `not` | Negates the condition |

### Advanced Conditions

**File Size Checks**:
```yara
condition:
    any of them and (filesize < 700KB)
```

**String Position**:
```yara
condition:
    $mz at 0 and $signature in (0..1024)
```

**String Count**:
```yara
condition:
    #malicious_string > 5
```

**Entry Point Check**:
```yara
condition:
    $code at entrypoint
```

---

## 📚 Real-World Example: IcedID Detection

The evil kingdom of Malhare deployed the **IcedID** trojan to steal credentials. TBFC's analysts discovered a common signature across infected systems.

### Creating the Detection Rule

```yara
rule TBFC_IcedID_Detect
{
    meta:
        author = "TBFC SOC L2"
        description = "Detects IcedID malware loaders"
        date = "2025-12-13"
        confidence = "low"
        reference = "Dark Kingdom Campaign"

    strings:
        $mz = { 4D 5A }                        // MZ header (PE file)
        $hex1 = { 48 8B ?? ?? 48 89 }          // Malicious binary fragment
        $s1 = "malhare" nocase                 // IOC string

    condition:
        all of them and filesize < 10485760   // < 10MB
}
```

### Running the Rule

Save the rule to `icedid_starter.yar` and execute:

```bash
yara -r icedid_starter.yar C:\
```

**Sample Output**:
```
icedid_starter  C:\Users\WarevilleElf\AppData\Roaming\TBFC_Presents\malhare_gift_loader.exe
```

### Enhanced Scan with String Display

```bash
yara -r -s icedid_starter.yar C:\
```

This shows **which strings matched** in each file, providing valuable context for analysis.

---

## 🎯 Practical Challenge Walkthrough

### Challenge Objective

The blue team needs to:
1. Search for the keyword `TBFC:` followed by alphanumeric characters
2. Scan the `/home/ubuntu/Downloads/easter` directory
3. Extract code words from matching files
4. Decode McSkidy's hidden message

### 🔧 Step 1: Understand the Pattern

We're looking for strings matching this pattern:
- Starts with `TBFC:`
- Followed by one or more alphanumeric ASCII characters
- Regex pattern: `TBFC:[A-Za-z0-9]+`

**Pattern Breakdown**:
```regex
/TBFC:[A-Za-z0-9]+/
  ^    ^        ^
  |    |        |
  |    |        One or more alphanumeric characters
  |    Character class (letters and digits)
  Literal text to match
```

### 🔧 Step 2: Create the YARA Rule

Create a file named `mckidy_message.yar`:

```yara
rule TBFC_Message_Extractor
{
    meta:
        author = "TBFC Blue Team"
        description = "Extracts McSkidy's hidden message from images"
        date = "2025-12-13"
        mission = "Decode crisis communication"
    
    strings:
        $message = /TBFC:[A-Za-z0-9]+/
    
    condition:
        $message
}
```

### 🔧 Step 3: Execute the YARA Rule

Run the rule against the easter directory:

```bash
cd /home/ubuntu/Downloads
yara -r -s mckidy_message.yar easter/
```

**Command Breakdown**:
- `yara` - YARA command
- `-r` - Recursive scan (scan subdirectories)
- `-s` - Show matching strings
- `mckidy_message.yar` - Your rule file
- `easter/` - Target directory

### 🔧 Step 4: Analyze the Output

When executed, YARA will display something like:

```
TBFC_Message_Extractor easter/image1.png
0x1a2b:$message: TBFC:Find

TBFC_Message_Extractor easter/image2.png
0x3c4d:$message: TBFC:me

TBFC_Message_Extractor easter/image3.png
0x5e6f:$message: TBFC:in

TBFC_Message_Extractor easter/image4.png
0x7g8h:$message: TBFC:HopSec

TBFC_Message_Extractor easter/image5.png
0x9i0j:$message: TBFC:Island
```

### 🔧 Step 5: Extract Code Words

From each matched string, extract the word after `TBFC:`:

| File | Offset | Code Word |
|------|--------|-----------|
| image1.png | 0x1a2b | **Find** |
| image2.png | 0x3c4d | **me** |
| image3.png | 0x5e6f | **in** |
| image4.png | 0x7g8h | **HopSec** |
| image5.png | 0x9i0j | **Island** |

### 🔧 Step 6: Decode the Message

Combine the code words in order:

**McSkidy's Hidden Message**: `Find me in HopSec Island`

🎉 **Success!** You've decoded McSkidy's location using YARA pattern matching!

---

## 🚩 Challenge Solutions & Flags

### Question 1: How many images contain the string TBFC?

**Answer**: `5`

**Explanation**: 
When you run the YARA rule with the `-r` flag against the `/home/ubuntu/Downloads/easter` directory, it recursively scans all files and detects 5 image files containing strings matching the pattern `TBFC:[A-Za-z0-9]+`.

**Verification**:
```bash
yara -r mckidy_message.yar easter/ | wc -l
# Output: 5
```

---

### Question 2: What regex would you use to match a string that begins with TBFC: followed by one or more alphanumeric ASCII characters?

**Answer**: `/TBFC:[A-Za-z0-9]+/`

**Component Breakdown**:

| Component | Purpose |
|-----------|---------|
| `/` | Regex delimiter (start) |
| `TBFC:` | Literal text to match exactly |
| `[A-Za-z0-9]` | Character class (any letter or digit) |
| `+` | Quantifier (one or more) |
| `/` | Regex delimiter (end) |

**Why This Works**:
- Matches `TBFC:Find`, `TBFC:me`, `TBFC:HopSec`, etc.
- Won't match `TBFC:` alone (requires at least one character)
- Won't match `TBFC` without colon
- Won't match special characters after colon

---

### Question 3: What is the message sent by McSkidy?

**Answer**: `Find me in HopSec Island`

**Decoding Process**:

1. **Scan Results**:
   ```
   TBFC:Find     (from image1.png)
   TBFC:me       (from image2.png)
   TBFC:in       (from image3.png)
   TBFC:HopSec   (from image4.png)
   TBFC:Island   (from image5.png)
   ```

2. **Extract Code Words**:
   - Find
   - me
   - in
   - HopSec
   - Island

3. **Combine in Order**:
   - "Find me in HopSec Island"

**Intelligence Value**: 
This decoded message reveals McSkidy's location, providing the blue team with crucial intelligence to:
- Plan a rescue operation
- Understand McSkidy's current situation
- Continue the mission to save SOC-mas

---

## 🔧 Essential YARA Commands Reference

### Basic Operations

| Command | Description | Use Case |
|---------|-------------|----------|
| `yara rule.yar file` | Scan single file | Quick file check |
| `yara -r rule.yar dir/` | Recursive scan | Scan entire directory tree |
| `yara -s rule.yar file` | Show matching strings | See what triggered the rule |
| `yara -g rule.yar file` | Print tags | Display rule tags |
| `yara -m rule.yar file` | Print metadata | Show rule information |

### Advanced Options

| Command | Description | Use Case |
|---------|-------------|----------|
| `yara -w rule.yar file` | Disable warnings | Clean output in scripts |
| `yara -n rule.yar file` | Only print not satisfied rules | Debugging rule logic |
| `yara -d var=value rule.yar file` | Define external variable | Dynamic rule behavior |
| `yara -p threads rule.yar dir/` | Parallel scanning | Speed up large scans |
| `yara -f rule.yar file` | Fast matching mode | Quick initial triage |

### Output Control

```bash
# Count matches
yara -r rule.yar dir/ | wc -l

# Save results to file
yara -r -s rule.yar dir/ > results.txt

# Search for specific matches
yara -r rule.yar dir/ | grep "malicious"

# Formatted output with metadata
yara -r -m -s rule.yar dir/ > detailed_report.txt
```

---

## 💡 YARA Best Practices

### 1. 📋 Metadata is Your Friend

Always include comprehensive metadata:

```yara
meta:
    author = "Your Name"
    description = "Clear description of what this detects"
    date = "2025-12-13"
    version = "1.0"
    reference = "https://link-to-threat-intel"
    confidence = "high|medium|low"
    severity = "critical|high|medium|low"
    tlp = "white|green|amber|red"
```

### 2. 🎯 Start Simple, Then Refine

**Bad Approach**:
```yara
// Overly complex first attempt
rule Complex_Rule
{
    strings:
        $s1 = { 48 [10-20] 89 ?? [5-10] C3 }
        $s2 = /complicated.*regex.*pattern/
    condition:
        ($s1 and #s2 > 5) or (filesize < 1MB and entrypoint > 0x1000)
}
```

**Good Approach**:
```yara
// Start simple
rule Simple_Rule_v1
{
    strings:
        $s1 = "malware_string"
    condition:
        $s1
}

// Then add complexity based on testing
rule Simple_Rule_v2
{
    strings:
        $s1 = "malware_string"
        $s2 = { 4D 5A }
    condition:
        $s1 and $s2 and filesize < 1MB
}
```

### 3. 🧪 Test Against Known Samples

Create a test directory structure:

```
test_samples/
├── malicious/
│   ├── sample1.exe
│   ├── sample2.dll
│   └── sample3.bin
└── benign/
    ├── clean1.exe
    ├── clean2.dll
    └── clean3.bin
```

Test your rule:
```bash
# Should detect malicious samples
yara -r rule.yar test_samples/malicious/

# Should NOT detect benign samples
yara -r rule.yar test_samples/benign/
```

### 4. 🔄 Use String Sets Effectively

```yara
rule Organized_Strings
{
    strings:
        // API calls
        $api1 = "CreateRemoteThread" ascii
        $api2 = "VirtualAllocEx" ascii
        
        // File paths
        $path1 = "\\AppData\\Roaming\\" nocase
        $path2 = "\\Temp\\" nocase
        
        // Network indicators
        $url1 = /http:\/\/\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}/
    
    condition:
        2 of ($api*) and any of ($path*) and $url1
}
```

### 5. ⚡ Optimize for Performance

**Slow Rule** (avoid):
```yara
strings:
    $slow = /.*malware.*/ wide ascii
```

**Optimized Rule**:
```yara
strings:
    $fast = "malware" wide ascii nocase
```

### 6. 📝 Comment Complex Patterns

```yara
rule Well_Documented
{
    strings:
        // Detects XOR-encoded "Malhare" with common XOR keys
        $xor1 = "Malhare" xor (0x00-0xFF)
        
        // PE header check - ensures we're scanning executables
        $mz = { 4D 5A }
        
        // Suspicious API call sequence indicating process injection
        $inject = { 48 8B ?? ?? 48 89 [2-4] E8 }
    
    condition:
        $mz at 0 and ($xor1 or $inject)
}
```

### 7. 🔄 Version and Update Rules

```yara
rule Threat_Detection_v2
{
    meta:
        version = "2.0"
        changelog = "Added XOR detection, improved hex pattern"
        supersedes = "Threat_Detection_v1"
    
    strings:
        // ... updated strings
    
    condition:
        // ... improved conditions
}
```

---

## 🎓 Key Takeaways

1. **🎯 YARA is Pattern Matching at Its Best**
   - Create custom detection rules based on observable patterns
   - No waiting for vendor signatures or updates

2. **📝 Three Core Components**
   - **Metadata**: Who, what, when, why
   - **Strings**: What to search for
   - **Conditions**: When to trigger

3. **🔤 Three String Types**
   - **Text**: Human-readable strings
   - **Hex**: Binary byte patterns
   - **Regex**: Flexible pattern matching

4. **⚖️ Conditions Define Logic**
   - Simple: `any of them`, `all of them`
   - Complex: Boolean operators, file properties
   - Advanced: String counts, positions, entropy

5. **🤝 Community Strength**
   - Share rules to improve collective defense
   - Leverage existing rules from GitHub, forums
   - Contribute back to the community

6. **🔍 Practical Applications**
   - Malware detection across endpoints
   - Threat hunting in memory and disk
   - Intelligence extraction from files
   - Post-incident analysis and forensics

7. **🎯 McSkidy's Mission Complete**
   - Decoded message: "Find me in HopSec Island"
   - Demonstrates YARA's versatility beyond malware
   - Pattern matching solves diverse security challenges

---

## 🔗 Related TryHackMe Rooms

Continue your YARA journey:

| Room | Focus Area | Difficulty | Duration |
|------|-----------|-----------|----------|
| **Yara** | In-depth YARA rule writing | Easy | 45 min |
| **Malware Analysis** | Analyzing malicious samples | Medium | 90 min |
| **Threat Hunting** | Proactive threat detection | Medium | 120 min |
| **Memory Forensics** | Analyzing memory dumps | Hard | 180 min |
| **Volatility** | Memory analysis with YARA | Medium | 90 min |

---

## 📚 Additional Resources

### 📖 Documentation
- [YARA Official Docs](https://yara.readthedocs.io/) - Complete reference
- [YARA GitHub](https://github.com/VirusTotal/yara) - Source code and issues
- [YARA-Rules Repository](https://github.com/Yara-Rules/rules) - Community rules
- [Awesome YARA](https://github.com/InQuest/awesome-yara) - Curated resources

### 🛠️ Tools & Utilities

| Tool | Purpose | Link |
|------|---------|------|
| **yarGen** | Automatic rule generator | [GitHub](https://github.com/Neo23x0/yarGen) |
| **YaraGuardian** | Web interface for rule management | [GitHub](https://github.com/PUNCH-Cyber/YaraGuardian) |
| **Loki** | IOC and YARA scanner | [GitHub](https://github.com/Neo23x0/Loki) |
| **YARA-X** | Next-gen YARA implementation | [GitHub](https://github.com/VirusTotal/yara-x) |
| **yaraQA** | Rule quality assurance | [GitHub](https://github.com/JPCERTCC/yaraQA) |

### 📝 Learning Resources
- [YARA in a Nutshell](https://blog.nviso.eu/2021/06/23/yara-in-a-nutshell/)
- [Writing YARA Rules (SANS)](https://www.sans.org/reading-room/whitepapers/forensics/writing-yara-rules-fun-profit-39395)
- [YARA Forge](https://yaraify.abuse.ch/) - Online YARA rule tester

### 🎓 Advanced Topics
- [YARA Module Development](https://yara.readthedocs.io/en/latest/modules.html)
- [PE Module for Windows Analysis](https://yara.readthedocs.io/en/latest/modules/pe.html)
- [ELF Module for Linux Analysis](https://yara.readthedocs.io/en/latest/modules/elf.html)

---

## 🌐 Real-World YARA Usage

### Industry Applications

**1. 🏢 Endpoint Detection and Response (EDR)**
- Modern EDR solutions use YARA to detect malicious processes in real-time
- Rules scan memory and disk for known malware families
- Custom rules detect organization-specific threats

**2. 🧠 Threat Intelligence Platforms**
- Security teams share YARA rules as Indicators of Compromise (IOCs)
- Rules distributed via STIX/TAXII protocols
- Automated rule updates from threat feeds

**3. 🚨 Incident Response**
- IR teams deploy YARA to quickly scan compromised networks
- Hunt for lateral movement artifacts
- Identify all affected systems in minutes

**4. 🔬 Malware Research**
- Researchers classify and track malware families
- Identify relationships between samples
- Detect mutations and variants

**5. 🛡️ Network Security**
- IDS/IPS systems use YARA for deep packet inspection
- Scan network traffic for malicious payloads
- Detect C2 communication patterns

---

## 🤝 Contributing

Found an error or have suggestions? Contributions are welcome!

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/yara-improvement`)
3. Commit your changes (`git commit -am 'Add YARA rule optimization tips'`)
4. Push to the branch (`git push origin feature/yara-improvement`)
5. Open a Pull Request

---

## 📜 License

This walkthrough is provided for educational purposes as part of TryHackMe's Advent of Cyber 2025 challenge.

**Disclaimer**: All techniques described should only be used in authorized security testing environments and for legitimate threat hunting purposes.

---

## 🌟 Support

If this walkthrough helped you:
- ⭐ Star this repository

---

<div align="center">

###
