# 🎄 TryHackMe Advent of Cyber 2025 - Day 15 🎄

## Web Attack Forensics - Drone Alone: Command Injection Investigation with Splunk

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/room/adventofcyber2025)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-30%20minutes-blue?style=for-the-badge)](https://tryhackme.com)
[![Day](https://img.shields.io/badge/Day-15-orange?style=for-the-badge)](https://tryhackme.com)

---

## 📖 Story Overview

TBFC's drone scheduler web UI is experiencing suspicious activity. Long HTTP requests containing Base64-encoded chunks are flooding in. Splunk raises a critical alert:

**"Apache spawned an unusual process."**

On multiple endpoints, these requests are causing the web server to execute shell code—obfuscated and hidden within Base64 payloads. The attack targets vulnerable CGI scripts (`hello.bat`) to gain command execution on the underlying operating system.

**Your Mission**: As a Blue Team member working alongside elf Log McBlue, triage the incident, identify compromised hosts, extract and decode malicious payloads, and determine the full scope of the web attack.

You'll use **Splunk** to pivot between:
- 🌐 Web layer (Apache access/error logs)
- 💻 Host layer (Sysmon process telemetry)

Time to unravel the attack chain! 🕵️‍♂️🔍

---

## 🎯 Learning Objectives

By completing this challenge, you'll master:

- ✅ **Web Log Analysis** - Detecting malicious activity in Apache logs
- ✅ **Command Injection Detection** - Identifying exploitation attempts
- ✅ **OS-Level Investigation** - Using Sysmon for process tracking
- ✅ **Payload Decoding** - Decoding obfuscated Base64 commands
- ✅ **Attack Chain Reconstruction** - Piecing together the full narrative
- ✅ **Splunk SIEM** - Advanced queries for Blue Team investigations

---

## 🔍 Understanding Command Injection

**Command Injection** allows attackers to execute arbitrary OS commands on the server hosting a web application.

### Attack Flow

```
┌─────────────────────────────────────┐
│   1. Vulnerable Web Application     │
│      (CGI script with poor input    │
│       validation)                   │
└────────────┬────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│   2. Attacker Crafts Malicious      │
│      Input                          │
│      ?cmd=powershell -enc [base64]  │
└────────────┬────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│   3. Application Fails to Sanitize  │
│      Input                          │
└────────────┬────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│   4. OS Command Executed            │
│      Apache → cmd.exe → powershell  │
└────────────┬────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│   5. Attacker Gains Code Execution  │
│      System compromised!            │
└─────────────────────────────────────┘
```

### Common Injection Characters

| Character | Purpose | Example |
|-----------|---------|---------|
| `;` | Command separator | `input;whoami` |
| `&` | Background execution | `input&whoami` |
| `\|` | Pipe output | `input\|whoami` |
| `&&` | AND operator | `input&&whoami` |
| `\|\|` | OR operator | `input\|\|whoami` |
| `` ` `` | Command substitution | ``input`whoami` `` |
| `$()` | Command substitution | `input$(whoami)` |

---

## 🐳 Splunk Investigation Environment

### Access Credentials

```
URL:      http://MACHINE_IP:8000
Username: Blue
Password: Pass1234
```

### Critical Configuration

⚠️ **IMPORTANT**: Set time range to **"Last 7 days"** or **"All time"** in Splunk!

If the default range is too narrow, you'll see "No results found."

### Data Sources

| Index | Description | Event Types |
|-------|-------------|-------------|
| `windows_apache_access` | HTTP access logs | Requests, responses, client IPs |
| `windows_apache_error` | Server error logs | Execution failures, errors |
| `windows_sysmon` | Process monitoring | Process creation, network, file events |

---

## 🎯 Investigation Walkthrough

### 🔍 **Step 1: Detect Suspicious Web Commands**

**Goal**: Find HTTP requests showing command execution attempts

**Splunk Query:**
```spl
index=windows_apache_access (cmd.exe OR powershell OR "powershell.exe" OR "Invoke-Expression") 
| table _time host clientip uri_path uri_query status
```

**Query Breakdown:**

| Component | Purpose |
|-----------|---------|
| `index=windows_apache_access` | Search Apache web access logs |
| `(cmd.exe OR powershell ...)` | Look for command execution keywords |
| `\| table` | Format results with specific fields |

**What to Look For:**

✅ Base64-encoded strings in URI parameters
✅ Command execution attempts in query strings  
✅ Suspicious patterns like `/cgi-bin/hello.bat?cmd=...`

**Expected Results:**
```
_time                  host        clientip       uri_path           uri_query                     status
2024-12-15 10:23:45    drone-srv   192.168.1.100  /cgi-bin/hello.bat cmd=powershell -enc VABoA...  200
```

#### 🔓 Decoding Base64 Payloads

When you find Base64-encoded PowerShell commands, decode them!

**Example Encoded String:**
```
VABoAGkAcwAgAGkAcwAgAG4AbwB3ACAATQBpAG4AZQAhACAATQBVAEEASABBAEEASABBAEEA
```

**Decoding Method:**
1. Copy the Base64 string
2. Visit https://www.base64decode.org/
3. Paste in the decode field
4. Click **"DECODE"**

**Decoded Result:**
```
This is now Mine! MUAHAHAHA
```

**Analysis**: Confirms attacker's malicious message executed successfully! 🚨

---

### 🔥 **Step 2: Look for Server-Side Errors**

**Goal**: Find execution attempts or internal failures in error logs

**Splunk Query:**
```spl
index=windows_apache_error ("cmd.exe" OR "powershell" OR "Internal Server Error")
```

⚠️ **IMPORTANT**: Select **View: Raw** from dropdown above Event display field!

**What to Look For:**

✅ 500 Internal Server Error messages  
✅ Error messages mentioning cmd.exe or powershell  
✅ Failed execution attempts  
✅ CGI script errors  

**Why This Matters:**

If `/cgi-bin/hello.bat?cmd=powershell` triggers a **500 error**, it means:
1. ✅ Attacker's input was processed by server
2. ✅ Command attempted to execute
3. ✅ Execution may have failed, but attempt occurred

This is a **key indicator** that the attack reached the backend!

**Example Error Log:**
```
[Mon Dec 15 10:23:45 2024] [cgi:error] [pid 1234] [client 192.168.1.100:54321] 
AH01215: cmd.exe: 'powershell' is not recognized as an internal or external command
```

---

### 🔬 **Step 3: Trace Suspicious Process Creation**

**Goal**: Find malicious processes spawned by Apache

**Splunk Query:**
```spl
index=windows_sysmon ParentImage="*httpd.exe"
```

⚠️ **IMPORTANT**: Select **View: Table** from dropdown!

**What to Look For:**

**Normal Apache Behavior:**
- ✅ Spawns only worker threads
- ✅ No system process execution
- ✅ Stays within its process space

**Suspicious Activity:**
```
ParentImage: C:\Apache24\bin\httpd.exe
Image:       C:\Windows\System32\cmd.exe
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
             🚨 Apache should NEVER spawn cmd.exe!
```

**Example Results:**
```
ParentImage              Image                           CommandLine
C:\Apache24\bin\httpd    C:\Windows\System32\cmd.exe    cmd.exe /c whoami
C:\Apache24\bin\httpd    C:\Windows\System32\cmd.exe    cmd.exe /c powershell -enc ...
```

**Critical Finding:**

If Apache spawned system processes (`cmd.exe`, `powershell.exe`):
- 🚨 **Successful command injection**
- 🚨 **Web attack penetrated the OS**
- 🚨 **Attacker achieved code execution**

This is **one of the strongest indicators** the web attack succeeded!

---

### 🕵️ **Step 4: Confirm Attacker Enumeration**

**Goal**: Identify post-exploitation reconnaissance

**Splunk Query:**
```spl
index=windows_sysmon *cmd.exe* *whoami*
```

**Why "whoami" Matters:**

Attackers use `whoami` immediately after gaining code execution to:
1. ✅ Determine which user account is running
2. ✅ Assess privilege level
3. ✅ Plan next steps (privilege escalation, lateral movement)

**Example Results:**
```
Image:        C:\Windows\System32\whoami.exe
ParentImage:  C:\Windows\System32\cmd.exe
CommandLine:  whoami
User:         NT AUTHORITY\SYSTEM
```

**What This Confirms:**
- ✅ Attacker successfully executed reconnaissance
- ✅ Command injection worked end-to-end
- ✅ Attacker knows privilege level
- ✅ Post-exploitation phase has begun

---

### 🔐 **Step 5: Identify Encoded PowerShell Payloads**

**Goal**: Find all encoded command attempts

**Splunk Query:**
```spl
index=windows_sysmon Image="*powershell.exe" (CommandLine="*enc*" OR CommandLine="*-EncodedCommand*" OR CommandLine="*Base64*")
```

**What to Look For:**

**Encoded PowerShell Command:**
```
powershell.exe -enc VABoAGkAcwAgAGkAcwAgAG4AbwB3ACAATQBpAG4AZQAhACAATQBVAEEASABBAEEISABBAEEA
                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                    Base64-encoded command attempting to hide malicious intent
```

**Expected Results:**

If **defenses are working**: **No results** ✅
- Encoded payload never executed
- Attack blocked successfully

If **results appear**: 🚨
- Attack succeeded
- Decode the Base64 immediately
- Incident response required NOW

---

## 🚩 Challenge Solutions & Answers

### Question 1: What is the reconnaissance executable file name?

**Answer**: `whoami.exe`

**How to Find:**
```spl
index=windows_sysmon *cmd.exe* *whoami*
```

**Explanation**: 
The `whoami` command is a standard Windows utility that displays:
- Current username
- Security context
- User privileges

Attackers use it immediately after gaining code execution to understand:
- ✅ What user account they're running as
- ✅ What privileges they have
- ✅ Whether privilege escalation is needed

**Process Chain Found:**
```
Apache (httpd.exe)
  └─> cmd.exe
       └─> whoami.exe  ← Reconnaissance!
```

This confirms post-exploitation reconnaissance activity.

---

### Question 2: What executable did the attacker attempt to run through the command injection?

**Answer**: `PowerShell.exe`

**How to Find:**
```spl
index=windows_apache_access (cmd.exe OR powershell OR "powershell.exe" OR "Invoke-Expression")
```

And:
```spl
index=windows_sysmon ParentImage="*httpd.exe"
```

**Explanation**: 
PowerShell is a powerful scripting framework. Attackers favor it because:
- ✅ Built into all modern Windows systems
- ✅ Can download and execute payloads
- ✅ Supports Base64-encoded commands (obfuscation)
- ✅ Can bypass some security controls
- ✅ "Living off the land" - no malware upload needed

**Attack Flow:**
```
Web Request → CGI Script → cmd.exe → PowerShell.exe → Malicious Payload
```

Both Splunk queries revealed PowerShell.exe being invoked through command injection, with Apache as the parent process—confirming successful exploitation.

---

## 🔗 Complete Attack Chain Reconstruction

### Phase 1: Initial Access (Web Layer)

```
┌─────────────────────────────────────────────────────────┐
│ Attacker sends malicious HTTP request                   │
│ GET /cgi-bin/hello.bat?cmd=powershell -enc [BASE64]    │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ Apache processes vulnerable CGI script                   │
│ Input validation missing/inadequate                      │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ Command injection successful!                            │
│ Arbitrary OS command execution achieved                  │
└─────────────────────────────────────────────────────────┘
```

### Phase 2: Command Execution (OS Layer)

```
┌─────────────────────────────────────────────────────────┐
│ Apache (httpd.exe) spawns cmd.exe                       │
│ Process relationship: httpd.exe → cmd.exe               │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ cmd.exe executes: whoami                                │
│ Attacker learns: Running as SYSTEM/apache user          │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ cmd.exe executes: powershell -enc [PAYLOAD]            │
│ Base64 payload decoded and executed                     │
└─────────────────────────────────────────────────────────┘
```

### Phase 3: Post-Exploitation

```
┌─────────────────────────────────────────────────────────┐
│ Decoded payload reveals:                                │
│ "This is now Mine! MUAHAHAHA"                           │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ Successful compromise confirmed                          │
│ System under attacker control                           │
│ Immediate incident response required                    │
└─────────────────────────────────────────────────────────┘
```

---

## 📊 Evidence Summary

| Investigation Step | Evidence Source | Key Finding | Severity |
|-------------------|----------------|-------------|----------|
| **1. Attack Detection** | Apache Access Logs | Suspicious URI with cmd.exe/powershell | 🔴 High |
| **2. Execution Errors** | Apache Error Logs | 500 errors, execution attempts | 🟠 Medium |
| **3. Process Creation** | Sysmon Logs | Apache spawned cmd.exe (abnormal!) | 🔴 Critical |
| **4. Reconnaissance** | Sysmon Logs | whoami.exe executed | 🟠 High |
| **5. Payload Delivery** | Sysmon Logs | PowerShell with Base64 encoding | 🔴 Critical |

---

## 🛡️ Detection & Prevention

### Blue Team Detection Rules

**1. Alert on Apache Spawning System Processes**
```spl
index=windows_sysmon ParentImage="*httpd.exe" 
(Image="*cmd.exe" OR Image="*powershell.exe")
| eval severity="CRITICAL"
| alert
```

**2. Alert on Encoded PowerShell Commands**
```spl
index=windows_sysmon Image="*powershell.exe" 
CommandLine="*-enc*"
| eval severity="HIGH"
| alert
```

**3. Alert on Suspicious Web Requests**
```spl
index=windows_apache_access 
(uri_query="*cmd.exe*" OR uri_query="*powershell*" OR uri_query="*whoami*")
| eval severity="HIGH"
| alert
```

**4. Alert on Reconnaissance Commands**
```spl
index=windows_sysmon 
(Image="*whoami.exe" OR Image="*ipconfig.exe" OR Image="*net.exe")
ParentImage="*cmd.exe"
| eval severity="MEDIUM"
| alert
```

---

### Hardening Measures

#### 1. Input Validation

```python
# Whitelist approach - SECURE
import re

allowed_pattern = re.compile(r'^[a-zA-Z0-9_]+$')

def validate_input(user_input):
    if not allowed_pattern.match(user_input):
        raise ValueError("Invalid input detected")
    return user_input
```

#### 2. Disable CGI Scripts

```apache
# Apache configuration
# Comment out CGI module
# LoadModule cgi_module modules/mod_cgi.so

# Or restrict CGI execution
<Directory "/var/www/cgi-bin">
    Options -ExecCGI
</Directory>
```

#### 3. Least Privilege

```apache
# Run Apache as non-privileged user
User apache
Group apache

# Limit capabilities
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
```

#### 4. Web Application Firewall (WAF)

```
# ModSecurity rules
SecRule ARGS "@rx (cmd\.exe|powershell|whoami|net\.exe)" \
    "id:1001,phase:2,deny,status:403,msg:'Command injection attempt detected'"

SecRule ARGS "@rx -enc.*[A-Za-z0-9+/=]{50,}" \
    "id:1002,phase:2,deny,status:403,msg:'Base64-encoded PowerShell detected'"
```

#### 5. PowerShell Security

```powershell
# Enable constrained language mode
$ExecutionContext.SessionState.LanguageMode = "ConstrainedLanguage"

# Enable script block logging
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
    -Name "EnableScriptBlockLogging" -Value 1

# Enable transcription
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription" `
    -Name "EnableTranscripting" -Value 1
```

---

## 🔧 Splunk Query Reference

### Essential SPL Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `index=` | Specify data source | `index=windows_sysmon` |
| `table` | Display fields as table | `\| table _time host user` |
| `stats` | Statistical operations | `\| stats count by clientip` |
| `where` | Filter results | `\| where status=500` |
| `eval` | Create calculated fields | `\| eval severity="HIGH"` |
| `timechart` | Time-based visualization | `\| timechart count` |
| `dedup` | Remove duplicates | `\| dedup clientip` |
| `sort` | Order results | `\| sort -_time` |
| `rex` | Extract fields with regex | `\| rex field=uri "(?<param>.*)"` |

### Investigation Query Templates

**Find Failed Login Attempts:**
```spl
index=windows_security EventCode=4625 
| stats count by Account_Name src_ip
| where count > 5
```

**Track Process Execution Timeline:**
```spl
index=windows_sysmon EventCode=1 
| table _time ParentImage Image CommandLine User
| sort _time
```

**Identify Lateral Movement:**
```spl
index=windows_sysmon EventCode=3 
(DestinationPort=445 OR DestinationPort=3389)
| stats count by SourceIp DestinationIp DestinationPort
```

**Detect PowerShell Download Cradles:**
```spl
index=windows_sysmon Image="*powershell.exe" 
(CommandLine="*downloadstring*" OR CommandLine="*invoke-webrequest*" OR CommandLine="*wget*")
| table _time User CommandLine
```

**Find Suspicious Network Connections:**
```spl
index=windows_sysmon EventCode=3 
NOT (DestinationPort=80 OR DestinationPort=443)
| stats count by Image DestinationIp DestinationPort
```

---

## 🎓 Key Takeaways

1. **🌐 Web Logs Reveal Initial Access**
   - Apache access logs show attack attempts
   - Error logs confirm execution attempts
   - Correlation is essential for full picture

2. **🔬 Process Relationships Expose Exploitation**
   - Web servers should never spawn system commands
   - Parent-child relationships reveal successful attacks
   - Sysmon provides invaluable telemetry

3. **🔐 Base64 is Obfuscation, Not Security**
   - Encoding ≠ Encryption
   - Attackers use Base64 to evade simple detection
   - Always decode suspicious strings

4. **🕵️ Reconnaissance Commands Signal Compromise**
   - `whoami`, `ipconfig`, `net user` indicate breach
   - Post-exploitation follows successful injection
   - Quick detection prevents further damage

5. **📊 SIEM Correlation is Critical**
   - Single log source = partial picture
   - Multiple sources = complete attack narrative
   - Splunk enables powerful cross-source analysis

6. **🛡️ Defense Requires Multiple Layers**
   - Application: Input validation
   - Network: WAF rules
   - Host: Endpoint protection
   - Detection: Logging and monitoring

7. **⚡ Detection Speed Matters**
   - Faster detection = less dwell time
   - Automated alerts enable rapid response
   - Practice makes perfect in IR

---

## 🔗 Related TryHackMe Rooms

Continue your Blue Team journey:

| Room | Focus Area | Difficulty | Duration |
|------|-----------|-----------|----------|
| **Splunk 101** | SIEM fundamentals | Easy | 60 min |
| **Splunk 2** | Advanced SPL queries | Medium | 90 min |
| **Investigating with Splunk** | Incident investigation | Medium | 90 min |
| **Sysmon** | Windows event logging | Easy | 45 min |
| **Web Application Security** | Common web vulnerabilities | Medium | 120 min |
| **Intrusion Detection** | Network monitoring | Medium | 90 min |

---

## 📚 Additional Resources

### 📖 Documentation & Standards
- [Splunk Search Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference) - Complete SPL documentation
- [Apache Log Format](https://httpd.apache.org/docs/current/logs.html) - Understanding Apache logs
- [Sysmon Configuration](https://github.com/SwiftOnSecurity/sysmon-config) - Industry-standard Sysmon config
- [OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection) - Vulnerability details

### 🛠️ Tools & Utilities

| Tool | Purpose | Link |
|------|---------|------|
| **CyberChef** | Data manipulation Swiss army knife | [Website](https://gchq.github.io/CyberChef/) |
| **Base64 Decode** | Quick Base64 decoding | [Website](https://www.base64decode.org/) |
| **Sysmon** | Windows system monitoring | [Microsoft](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon) |
| **Splunk** | SIEM platform | [Website](https://www.splunk.com/) |
| **ModSecurity** | Web application firewall | [GitHub](https://github.com/SpiderLabs/ModSecurity) |

### 📚 Books & Learning
- **"Practical Splunk Search Processing Language" by Ogunleye** - SPL mastery
- **"Blue Team Handbook" by Don Murdoch** - Incident response guide
- **"The Web Application Hacker's Handbook" by Stuttard & Pinto** - Understanding web attacks

### 🎥 Video Resources
- [Splunk Fundamentals](https://www.splunk.com/en_us/training.html) - Official training
- [Command Injection Explained](https://www.youtube.com/watch?v=pLIVLPF-Nd0) - Video tutorial

---


## 🤝 Contributing

Found an error or have suggestions? Contributions are welcome!

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/splunk-improvements`)
3. Commit your changes (`git commit -am 'Add Splunk query optimization'`)
4. Push to the branch (`git push origin feature/splunk-improvements`)
5. Open a Pull Request

---

## 📜 License

This walkthrough is provided for educational purposes as part of TryHackMe's Advent of Cyber 2025 challenge.

**Disclaimer**: All techniques described should only be used in authorized security testing environments and for legitimate Blue Team defense purposes.

---

## 🌟 Support

If this walkthrough helped you:
- ⭐ Star this repository


---

<div align="center">

### 🎄 Logs Don't Lie,Learn to Read Them! 🎄

**Detect. Investigate. Respond. Protect.**


![TryHackMe](https://tryhackme.com/img/logo/tryhackme_logo_full.svg)

</div>

---

## Tags

`#TryHackMe` `#AdventOfCyber2025` `#Splunk` `#SIEM` `#WebForensics` `#CommandInjection` `#LogAnalysis` `#BlueTeam` `#IncidentResponse` `#CyberSecurity` `#DFIR` `#Sysmon` `#ApacheLogs`
