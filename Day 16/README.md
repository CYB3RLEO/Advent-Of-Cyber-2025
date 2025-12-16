# 🎄 TryHackMe Advent of Cyber 2025 - Day 16 🎄

## Forensics - Registry Furensics: Investigating Windows Registry for Malware Evidence

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/room/adventofcyber2025)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-60%20minutes-blue?style=for-the-badge)](https://tryhackme.com)
[![Day](https://img.shields.io/badge/Day-16-orange?style=for-the-badge)](https://tryhackme.com)

---

## 📖 Story Overview

TBFC is under siege. Systems exhibit weird behavior, and the company deeply feels McSkidy's absence. However, McSkidy ensured her legacy would continue.

Her team, determined and well-trained, is confident in securing all systems and regaining control before SOC-mas. They've decided to conduct detailed forensic analysis on one of TBFC's most critical systems: **dispatch-srv01**.

### The Mission

The **dispatch-srv01** server coordinates drone-based gift delivery during SOC-mas. Recently, it was compromised by King Malhare's bandits of bunnies, threatening the entire delivery operation.

TBFC's defenders split into specialized teams for comprehensive forensics. While some investigate logs, memory dumps, and file systems, **you'll investigate the Windows Registry** of this compromised system.

**Abnormal Activity Started**: October 21st, 2025  
**Your Task**: Uncover evidence of compromise through registry forensics! 🔍💾

---

## 🎯 Learning Objectives

By completing this challenge, you'll master:

- ✅ **Windows Registry Fundamentals** - Understanding the "brain" of Windows
- ✅ **Registry Architecture** - Hives, Root Keys, and organization
- ✅ **Registry Editor** - Navigating the built-in tool
- ✅ **Registry Explorer** - Professional forensics analysis
- ✅ **Evidence Extraction** - Finding malware artifacts
- ✅ **Persistence Mechanisms** - Identifying autostart techniques

---

## 🧠 Understanding the Windows Registry

### The Windows "Brain"

Just like your brain stores everything you need to function (habits, preferences, routines, memories), **Windows needs a "brain" to store all its configurations**. This brain is the **Windows Registry**.

**Registry Definition**: A hierarchical database storing:
- Low-level OS settings
- Hardware configurations
- Software settings
- User preferences
- System services

---

## 📁 Registry Hives: The Building Blocks

Unlike a human brain in one location, the Windows Registry is spread across multiple files called **Hives**.

### Registry Hives Overview

```
Windows Registry
├── SYSTEM (Hardware, Drivers, Services)
├── SECURITY (Policies, Audit Settings)
├── SOFTWARE (Installed Programs, Autostarts)
├── SAM (User Accounts, Password Hashes)
├── NTUSER.DAT (User Preferences, Recent Files)
└── USRCLASS.DAT (Shellbags, Jump Lists)
```

### Detailed Hives Table

| Hive | Contains | Location |
|------|----------|----------|
| **SYSTEM** | Services, Drivers, Boot Config, Hardware | `C:\Windows\System32\config\SYSTEM` |
| **SECURITY** | Security Policies, Audit Settings | `C:\Windows\System32\config\SECURITY` |
| **SOFTWARE** | Programs, OS Version, Autostarts | `C:\Windows\System32\config\SOFTWARE` |
| **SAM** | Usernames, Password Hashes, Groups | `C:\Windows\System32\config\SAM` |
| **NTUSER.DAT** | User Preferences, Recent Files | `C:\Users\<username>\NTUSER.DAT` |
| **USRCLASS.DAT** | Shellbags, Jump Lists | `C:\Users\<username>\AppData\Local\Microsoft\Windows\USRCLASS.DAT` |

### Why Can't You Just Open Hive Files?

**Problem**: Hive files contain **binary data**—double-clicking them shows unreadable gibberish.

**Solution**: Use specialized tools (Registry Editor for live systems, Registry Explorer for forensics).

---

## 🔑 Registry Root Keys

Windows organizes hives into **Root Keys** visible in Registry Editor:

```
Computer
├── HKEY_CLASSES_ROOT (HKCR)
├── HKEY_CURRENT_USER (HKCU)
├── HKEY_LOCAL_MACHINE (HKLM)
│   ├── SYSTEM
│   ├── SECURITY
│   ├── SOFTWARE
│   └── SAM
├── HKEY_USERS (HKU)
└── HKEY_CURRENT_CONFIG (HKCC)
```

### Mapping: Hives to Root Keys

| Hive File | Registry Editor Location |
|-----------|-------------------------|
| SYSTEM | `HKEY_LOCAL_MACHINE\SYSTEM` |
| SECURITY | `HKEY_LOCAL_MACHINE\SECURITY` |
| SOFTWARE | `HKEY_LOCAL_MACHINE\SOFTWARE` |
| SAM | `HKEY_LOCAL_MACHINE\SAM` |
| NTUSER.DAT | `HKEY_CURRENT_USER` & `HKEY_USERS\<SID>` |
| USRCLASS.DAT | `HKEY_USERS\<SID>\Software\Classes` |

**Note**: HKCR and HKCC aren't separate files—they're dynamically generated when Windows runs.

---

## 🕵️ Registry Forensics: Why It Matters

**Registry Forensics** extracts and analyzes evidence from the registry during digital investigations.

### Forensically Important Registry Keys

| Key Path | Evidence Type |
|----------|--------------|
| `HKCU\...\Explorer\UserAssist` | Recently accessed GUI applications |
| `HKCU\...\Explorer\TypedPaths` | Paths typed in Explorer address bar |
| `HKLM\...\App Paths` | Application installation paths |
| `HKCU\...\Explorer\WordWheelQuery` | Search terms in Explorer |
| `HKLM\...\Run` | **Startup programs (PERSISTENCE!)** |
| `HKCU\...\Explorer\RecentDocs` | Recently accessed files |
| `HKLM\SYSTEM\...\ComputerName\ComputerName` | System hostname |
| `HKLM\SOFTWARE\...\Uninstall` | Installed applications |

---

## 🛠️ Registry Explorer vs Registry Editor

### Registry Editor (regedit.exe)

**Pros:**
- ✅ Built into Windows
- ✅ Easy to access
- ✅ Real-time system view

**Cons:**
- ❌ Cannot open offline hives
- ❌ Displays binary data (unreadable)
- ❌ Risk of modifying live system
- ❌ Limited forensic features

### Registry Explorer (Forensics Tool)

**Pros:**
- ✅ Opens offline hives
- ✅ Parses binary data
- ✅ Handles transaction logs
- ✅ Includes forensic bookmarks
- ✅ Advanced search capabilities
- ✅ No risk of evidence modification

**Cons:**
- ❌ Requires separate download
- ❌ Slight learning curve

**Verdict**: Use **Registry Explorer** for forensic investigations!

---

## 🎯 Practical Investigation Walkthrough

### Investigation Parameters

**Target**: dispatch-srv01  
**Incident Date**: October 21st, 2025  
**Hives Location**: `C:\Users\Administrator\Desktop\Registry Hives`  
**Tool**: Registry Explorer  

---

### 📋 **Step 1: Launch Registry Explorer**

Click the **Registry Explorer** icon on the taskbar.

---

### 📂 **Step 2: Load Registry Hives**

1. Click **File** → **Load hive**
2. Navigate to hives directory
3. Select a hive file

---

### ⚠️ **Step 3: Handle "Dirty" Hives (CRITICAL!)**

**What are "Dirty" Hives?**
- Hives collected from live systems may have incomplete transactions
- Missing transaction data = corrupted/inconsistent view

**Solution: Load with Transaction Logs**

**Process:**
1. Select hive file (e.g., SYSTEM)
2. **Hold SHIFT key**
3. Click **Open**
4. Transaction logs load automatically
5. See confirmation: "Transaction logs replayed successfully"

**Why This Matters**: Ensures accurate, consistent data for analysis.

---

### 🔍 **Step 4: Practice Navigation - Find Computer Name**

**Goal**: Verify system hostname

**Path**: `ROOT\ControlSet001\Control\ComputerName\ComputerName`

**Methods:**

**A) Manual Navigation:**
```
ROOT
└── ControlSet001
    └── Control
        └── ComputerName
            └── ComputerName (key)
                └── ComputerName (value) = DISPATCH-SRV01
```

**B) Search Bar:**
- Type: "ComputerName"
- Registry Explorer jumps to key

**C) Bookmarks:**
- Click "Available Bookmarks" tab
- Select "ComputerName" from forensic bookmarks

**Result**: Computer name = **DISPATCH-SRV01** ✅

---

## 🔎 Investigation Questions & Solutions

### 🎯 **Question 1: Installed Application**

**Question**: What application was installed on dispatch-srv01 before the abnormal activity started?

**Investigation Approach:**

**Registry Key**: `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall`

**Steps:**
1. Load **SOFTWARE** hive
2. Navigate to `ROOT\Microsoft\Windows\CurrentVersion\Uninstall`
3. Browse installed applications
4. Check DisplayName values
5. Look for installation dates around October 21st
6. Identify suspicious or drone-related software

**Search Method:**
- Use search bar: "Uninstall"
- Filter results by date
- Look for "Drone" or "Manager" keywords

**Answer**: `DroneManager Updater`

**Analysis:**
- Suspicious "updater" application
- Common malware disguise (users trust update tools)
- Installation timing matches incident start
- Drone-related name targeting dispatch system

---

### 🛤️ **Question 2: Application Execution Path**

**Question**: What is the full path where the user launched the application from?

**Investigation Approach:**

**Registry Keys to Check:**

1. `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`
2. `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU`
3. `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist`

**Steps:**
1. Load **NTUSER.DAT** hive
2. Navigate to execution history keys
3. Search for "DroneManager" references
4. Look for .exe paths in Downloads folder
5. Check Data/Value fields

**Search Method:**
- Search for: "DroneManager"
- Filter by executable (.exe)
- Check Downloads folder paths

**Answer**: `C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe`

**Analysis:**
- Executed from Downloads folder
- Indicates internet download
- Common initial infection vector
- User-initiated execution (social engineering success)

---

### 🔄 **Question 3: Persistence Mechanism**

**Question**: Which value was added by the application to maintain persistence on startup?

**Investigation Approach:**

**Registry Keys to Check:**

1. `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` (System-wide)
2. `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` (User-specific)

**Steps:**
1. Load **SOFTWARE** hive (for HKLM) or **NTUSER.DAT** (for HKCU)
2. Navigate to `ROOT\Microsoft\Windows\CurrentVersion\Run`
3. Look for recently added entries (October 21st timeframe)
4. Identify DroneManager-related values
5. Examine Data field for command and arguments

**What to Look For:**
- New entries created around incident date
- DroneManager or drone-related names
- Suspicious command-line flags:
  - `--background` (silent execution)
  - `--hidden` (concealment)
  - `--silent` (no user interaction)

**Answer**: `"C:\Program Files\DroneManager\dronehelper.exe" --background`

**Analysis:**
- **Persistence established**: Auto-starts on boot
- **Location**: Program Files (system-wide installation)
- **Executable**: dronehelper.exe (helper/service name - common disguise)
- **Flag**: --background (runs silently, no user awareness)
- **Impact**: Maintains attacker access after reboot

---

## 🔗 Complete Attack Chain Reconstruction

### Timeline Visualization

```
┌─────────────────────────────────────────────────────┐
│ October 21st, 2025 - Attack Begins                  │
└─────────────────────────────────────────────────────┘

Step 1: Initial Compromise
├─ User receives social engineering attack
├─ Downloads "DroneManager Updater"
└─ File location: C:\Users\dispatch.admin\Downloads\
                   ↓
Step 2: Execution
├─ User runs: DroneManager_Setup.exe
├─ Believes it's legitimate drone management update
└─ Installation begins
                   ↓
Step 3: Installation
├─ Malware extracts to: C:\Program Files\DroneManager\
├─ Creates: dronehelper.exe
└─ Appears as legitimate program
                   ↓
Step 4: Persistence Establishment
├─ Registry modification
├─ Key: HKLM\SOFTWARE\...\Run
├─ Value: "C:\Program Files\DroneManager\dronehelper.exe" --background
└─ Ensures auto-start on every boot
                   ↓
Step 5: System Compromise
├─ dronehelper.exe runs on startup
├─ Maintains backdoor access
├─ Compromises drone delivery system
└─ SOC-mas gift delivery at risk!
```

### Evidence Summary

| Evidence Type | Registry Location | Finding | Significance |
|---------------|-------------------|---------|--------------|
| **Installation** | `SOFTWARE\...\Uninstall` | DroneManager Updater | Malicious software installed |
| **Execution** | `NTUSER.DAT\...\RecentDocs` | Downloads\DroneManager_Setup.exe | Initial infection vector |
| **Persistence** | `SOFTWARE\...\Run` | dronehelper.exe --background | Maintains access post-reboot |
| **Target** | dispatch-srv01 | Drone delivery coordination | Critical infrastructure targeted |

---

## 🛡️ Defense & Detection Strategies

### Detection Indicators

**Registry-Based IOCs:**

```powershell
# Check for suspicious Run key entries
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"

# Look for recently installed programs
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" |
    Where-Object {$_.InstallDate -gt "20251020"}

# Audit user execution history
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU"
```

### Prevention Strategies

**1. Application Whitelisting**
```
# Only allow approved applications
# Block execution from Downloads, Temp folders
AppLocker / Windows Defender Application Control
```

**2. Monitor Registry Changes**
```
# Alert on Run key modifications
# Track software installation
# Monitor persistence mechanisms
Sysmon Event ID 13 (Registry value set)
```

**3. User Training**
```
# Recognize fake updaters
# Verify software sources
# Report suspicious downloads
# Don't trust email attachments
```

**4. Endpoint Protection**
```
# Real-time registry monitoring
# Behavioral analysis
# Heuristic detection
# Sandbox execution
```

---

## 🔧 Registry Forensics Command Reference

### PowerShell Registry Queries

```powershell
# List all startup programs
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Find recently modified registry keys
Get-ChildItem -Path "HKLM:\SOFTWARE" -Recurse |
    Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-7)}

# Export registry key to file
reg export "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" C:\export.reg

# Search registry for specific value
Get-ChildItem -Path "HKLM:\" -Recurse -ErrorAction SilentlyContinue |
    Get-ItemProperty | Where-Object {$_ -match "DroneManager"}

# List installed programs
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate
```

### Command Line Registry Tools

```cmd
# Query specific key
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Search for value
reg query HKLM /f "DroneManager" /s

# Export hive
reg save HKLM\SOFTWARE C:\forensics\SOFTWARE

# Compare registry snapshots
reg compare HKLM\SOFTWARE HKLM\SOFTWARE_BACKUP
```

---

## 📊 Forensic Analysis Workflow

### Standard Registry Investigation Process

```
1. Acquisition
   ├─ Collect hive files from compromised system
   ├─ Include transaction logs (.LOG, .LOG1, .LOG2)
   ├─ Hash files for integrity
   └─ Document collection process

2. Preparation
   ├─ Transfer to forensic workstation
   ├─ Verify file integrity (hash comparison)
   ├─ Make working copies
   └─ Never analyze originals

3. Analysis
   ├─ Load hives in Registry Explorer
   ├─ Apply transaction logs (SHIFT + Open)
   ├─ Navigate to forensic keys
   ├─ Search for IOCs
   └─ Document findings

4. Correlation
   ├─ Cross-reference with event logs
   ├─ Check file system timestamps
   ├─ Analyze prefetch data
   ├─ Review network logs
   └─ Build timeline

5. Reporting
   ├─ Summarize findings
   ├─ Include screenshots
   ├─ Provide recommendations
   └─ Maintain chain of custody
```

---

## 🎓 Key Takeaways

1. **🧠 Registry = Windows Brain**
   - Stores ALL system configurations
   - Organized into hives (physical files)
   - Mapped to root keys (logical structure)

2. **📁 Know Your Hives**
   - SYSTEM: Hardware, drivers, services
   - SOFTWARE: Installed programs, autostarts
   - SAM: User accounts, passwords
   - NTUSER.DAT: User-specific settings

3. **🔑 Persistence is Critical**
   - Malware establishes startup persistence
   - Run keys = most common method
   - Check both HKLM and HKCU

4. **🛠️ Tool Selection Matters**
   - Registry Editor: Live system only
   - Registry Explorer: Forensic analysis
   - Always use transaction logs

5. **🕵️ Multiple Evidence Sources**
   - Installation: Uninstall key
   - Execution: RecentDocs, UserAssist
   - Persistence: Run keys
   - Correlate across sources

6. **📝 Documentation Essential**
   - Screenshot everything
   - Record timestamps
   - Maintain investigation log
   - Preserve chain of custody

7. **🎯 Attack Pattern Recognition**
   - Downloads folder = common vector
   - "Updater" names = social engineering
   - --background flag = stealth execution
   - Run key = persistence goal

---

## 🔗 Related TryHackMe Rooms

Continue your forensics education:

| Room | Focus Area | Difficulty | Duration |
|------|-----------|-----------|----------|
| **Expediting Registry Analysis** | Advanced registry techniques | Medium | 90 min |
| **Windows Forensics 1** | File system artifacts | Medium | 90 min |
| **Windows Forensics 2** | Event log analysis | Medium | 90 min |
| **Autopsy** | Forensic analysis platform | Medium | 120 min |
| **Volatility** | Memory forensics | Hard | 120 min |
| **Redline** | Memory and file analysis | Medium | 90 min |

---

## 📚 Additional Resources

### 📖 Documentation & Standards
- [Registry Explorer by Eric Zimmerman](https://ericzimmerman.github.io/) - Official tool documentation
- [Windows Registry Forensics](https://www.sans.org/blog/registry-forensics/) - SANS comprehensive guide
- [Microsoft Registry Documentation](https://docs.microsoft.com/en-us/windows/win32/sysinfo/registry) - Official reference

### 🛠️ Forensic Tools Suite

| Tool | Purpose | Download |
|------|---------|----------|
| **Registry Explorer** | Primary forensic analysis | [Eric Zimmerman Tools](https://ericzimmerman.github.io/) |
| **RegRipper** | Automated registry parsing | [GitHub](https://github.com/keydet89/RegRipper3.0) |
| **RegistryChangesView** | Monitor live registry changes | [NirSoft](https://www.nirsoft.net/) |
| **Registry Viewer** | Alternative viewer | [AccessData](https://accessdata.com/) |

### 📚 Books & Courses
- **"Windows Registry Forensics" by Harlan Carvey** - The definitive guide
- **"Digital Forensics with Kali Linux" by Shiva V.** - Practical approach
- **SANS FOR500** - Windows Forensic Analysis course

### 🎥 Video Resources
- [13Cubed Registry Forensics Series](https://www.youtube.com/c/13cubed) - Excellent video tutorials
- [SANS DFIR Summit Talks](https://www.youtube.com/user/SANSForensics) - Expert presentations


---

## 🤝 Contributing

Found improvements or additional forensic techniques? Contributions welcome!

1. Fork the repository
2. Create feature branch (`git checkout -b feature/registry-improvements`)
3. Commit changes (`git commit -am 'Add registry forensic technique'`)
4. Push to branch (`git push origin feature/registry-improvements`)
5. Open Pull Request

---

## 📜 License

This walkthrough is provided for educational purposes as part of TryHackMe's Advent of Cyber 2025 challenge.

**Disclaimer**: Registry forensic techniques should only be used in authorized investigations and for legitimate security purposes.

---

## 🌟 Support

If this walkthrough helped you:
- ⭐ Star this repository


<div align="center">

### 🎄 The Registry Remembers Everything! 🎄

**Investigate. Analyze. Uncover. Protect.**

Made with ❤️ for the DFIR community

![TryHackMe](https://tryhackme.com/img/logo/tryhackme_logo_full.svg)

</div>

---

## Tags

`#TryHackMe` `#AdventOfCyber2025` `#RegistryForensics` `#WindowsForensics` `#DigitalForensics` `#DFIR` `#IncidentResponse` `#CyberSecurity` `#BlueTeam` `#MalwareAnalysis` `#ForensicAnalysis` `#RegistryExplorer`
