# 🎄 Advent of Cyber 2025 - Day 3: Splunk Basics - Did you SIEM?

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)
[![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-60%20min-blue?style=for-the-badge)](https://tryhackme.com)

## 📖 Overview

Day 3 introduces SIEM (Security Information and Event Management) analysis using Splunk. Investigate a sophisticated ransomware attack orchestrated by King Malhare's Bandit Bunnies against The Best Festival Company (TBFC).

### 🎯 Learning Objectives

- Ingest and interpret custom log data in Splunk
- Master Search Processing Language (SPL) queries
- Conduct multi-stage incident investigations
- Correlate evidence across multiple data sources
- Trace complete attack chains from recon to C2

## 🔍 Investigation Summary

**Threat Actor:** King Malhare & Bandit Bunnies (HopSec Island)  
**Target:** TBFC Web Server (`10.10.1.5`)  
**Attack Type:** Multi-stage ransomware deployment  
**Attacker IP:** `198.51.100.55`  
**Impact:** Ransomware deployment + Data exfiltration

## 📊 Data Sources

| Source Type | Description | Key Fields |
|-------------|-------------|------------|
| `web_traffic` | Web server access logs | `client_ip`, `user_agent`, `path`, `status` |
| `firewall_logs` | Network traffic logs | `src_ip`, `dest_ip`, `action`, `bytes_transferred` |

## 📝 Complete Walkthrough

### Step 1: Initial Data Exploration

```spl
# View all ingested logs
index=main

# Focus on web traffic
index=main sourcetype=web_traffic
```

### Step 2: Timeline Visualization

```spl
# Daily event count
index=main sourcetype=web_traffic | timechart span=1d count

# Sort by highest volume
index=main sourcetype=web_traffic | timechart span=1d count | sort by count | reverse
```

**Finding:** Peak traffic on **2025-10-12** 📈

### Step 3: Anomaly Detection

```spl
# Filter out legitimate browsers
index=main sourcetype=web_traffic user_agent!=*Mozilla* user_agent!=*Chrome* user_agent!=*Safari* user_agent!=*Firefox*
```

**Suspicious User Agents Found:**
- curl
- wget
- sqlmap
- Havij
- zgrab

### Step 4: Identify Attacker IP

```spl
# Top attacking IPs
sourcetype=web_traffic user_agent!=*Mozilla* user_agent!=*Chrome* user_agent!=*Safari* user_agent!=*Firefox* | stats count by client_ip | sort -count | head 5
```

**Primary Attacker:** `198.51.100.55` 🎯

### Step 5: Reconnaissance Phase

```spl
# Configuration file probing
sourcetype=web_traffic client_ip="198.51.100.55" AND path IN ("/.env", "/*phpinfo*", "/.git*") | table _time, path, user_agent, status
```

**Tactics:** Probing for exposed config files and version information

### Step 6: Vulnerability Testing

```spl
# Path traversal attempts
sourcetype=web_traffic client_ip="198.51.100.55" AND path="*..\/..\/*" OR path="*redirect*" | stats count by path
```

**Finding:** `658` path traversal attempts 🔓

### Step 7: SQL Injection Attack

```spl
# Identify SQL injection tools
sourcetype=web_traffic client_ip="198.51.100.55" AND user_agent IN ("*sqlmap*", "*Havij*") | table _time, path, status

# Count Havij events
sourcetype=web_traffic client_ip="198.51.100.55" user_agent="*Havij*" | stats count
```

**Finding:** `993` Havij user agent events 💉

### Step 8: Data Exfiltration

```spl
# Large file downloads
sourcetype=web_traffic client_ip="198.51.100.55" AND path IN ("*backup.zip*", "*logs.tar.gz*") | table _time, path, user_agent
```

**Tactics:** Stealing compressed archives

### Step 9: Remote Code Execution

```spl
# Webshell and ransomware execution
sourcetype=web_traffic client_ip="198.51.100.55" AND path IN ("*bunnylock.bin*", "*shell.php?cmd=*") | table _time, path, user_agent, status
```

**Critical Finding:** 🚨
- Webshell deployed: `shell.php`
- Ransomware executed: `bunnylock.bin`
- Full server compromise confirmed

### Step 10: C2 Communication

```spl
# Firewall correlation
sourcetype=firewall_logs src_ip="10.10.1.5" AND dest_ip="198.51.100.55" AND action="ALLOWED" | table _time, action, protocol, src_ip, dest_ip, dest_port, reason
```

**Finding:** Active C2 channel confirmed via firewall logs

### Step 11: Calculate Exfiltration Volume

```spl
# Sum bytes transferred
sourcetype=firewall_logs src_ip="10.10.1.5" AND dest_ip="198.51.100.55" AND action="ALLOWED" | stats sum(bytes_transferred) by src_ip
```

**Finding:** `126167` bytes exfiltrated 📤

## 🛠️ Essential SPL Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `index=` | Specify data source | `index=main` |
| `sourcetype=` | Filter by log type | `sourcetype=web_traffic` |
| `timechart` | Time-series visualization | `timechart span=1d count` |
| `stats` | Statistical aggregation | `stats count by field` |
| `table` | Display specific fields | `table _time, field1, field2` |
| `sort` | Order results | `sort -count` (descending) |
| `head` | Limit results | `head 10` |
| `IN()` | Multiple value matching | `field IN ("val1", "val2")` |
| `AND/OR` | Boolean logic | `field1="X" AND field2="Y"` |
| `!=` | Not equal | `user_agent!=*Mozilla*` |
| `*` | Wildcard | `path="*admin*"` |
| `sum()` | Calculate total | `stats sum(bytes) by ip` |

## 🎯 Attack Chain Breakdown

### Phase 1: Initial Access ⚔️
**Technique:** Reconnaissance  
**Tools:** curl, wget  
**Targets:** Config files, version info  
**Status Codes:** 403, 404, 401

### Phase 2: Exploitation 💥
**Technique:** SQL Injection  
**Tools:** sqlmap, Havij  
**Payload:** Time-based injection (SLEEP)  
**Indicator:** 504 Gateway Timeout

### Phase 3: Persistence 🔓
**Technique:** Webshell deployment  
**File:** shell.php  
**Capability:** Remote Code Execution (RCE)

### Phase 4: Payload Delivery 🦠
**Technique:** Ransomware staging  
**File:** bunnylock.bin  
**Method:** Executed via webshell

### Phase 5: C2 Communication 📡
**Source:** Compromised server (10.10.1.5)  
**Destination:** C2 server (198.51.100.55)  
**Evidence:** Firewall logs (ALLOWED, C2_CONTACT)

### Phase 6: Impact 💣
**Data Exfiltration:** 126,167 bytes  
**Ransomware:** Successfully deployed  
**Outcome:** Double extortion threat

## 🏆 Challenge Answers

| Question | Answer |
|----------|--------|
| Attacker IP? | `198.51.100.55` |
| Peak traffic day? (YYYY-MM-DD) | `2025-10-12` |
| Havij user_agent count? | `993` |
| Path traversal attempts? | `658` |
| Bytes transferred to C2? | `126167` |

## 🔐 Security Insights

### SIEM Best Practices
✅ **Establish baselines** for normal activity  
✅ **Correlate multiple sources** (web + firewall logs)  
✅ **Use time-series analysis** to spot spikes  
✅ **Filter progressively** from broad to specific  
✅ **Document IOCs** throughout investigation

### Attack Indicators (IOCs)

**Network Indicators:**
- IP: `198.51.100.55`
- Suspicious user agents: sqlmap, Havij, curl, wget, zgrab

**File Indicators:**
- `shell.php` (webshell)
- `bunnylock.bin` (ransomware)
- `logs.tar.gz` (exfiltrated data)

**Behavioral Indicators:**
- Path traversal patterns (`../../`)
- SQL injection payloads (`SLEEP(5)`)
- Config file enumeration (`/.env`, `/.git`)
- Outbound C2 communication

## 📚 Real-World Application

This investigation mirrors actual SOC workflows:

1. **Alert Triage:** Identify anomalous traffic spikes
2. **Scoping:** Determine affected systems and timeframe
3. **Evidence Collection:** Gather logs from multiple sources
4. **Timeline Analysis:** Reconstruct attacker actions
5. **Impact Assessment:** Quantify data loss and system compromise
6. **IOC Extraction:** Document indicators for blocking/hunting

## 🎓 Key Takeaways

- **SIEM is essential** for correlating security events
- **SPL proficiency** enables rapid investigation
- **Multi-stage attacks** follow predictable patterns (Kill Chain)
- **Log correlation** reveals the complete picture
- **Timeline visualization** identifies attack windows
- **Statistical analysis** highlights anomalies

## 🔗 Related Rooms

- [Incident Handling With Splunk](https://tryhackme.com)
- [Investigating with Splunk](https://tryhackme.com)
- [Splunk: Exploring SPL](https://tryhackme.com)
- [Intro to Log Analysis](https://tryhackme.com)

## 📖 Additional Resources

- [Splunk SPL Documentation](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [SANS Hunt Evil Poster](https://www.sans.org/posters/hunt-evil/)

## 🤝 Contributing

Found an issue or have improvements? Open an issue or PR!

## 📜 License

Educational purposes only. All credit to TryHackMe and Advent of Cyber 2025.

---

⭐ **If this helped you, drop a star!**

🎄 **Day 3/25 Complete - Keep hunting!**

[![Follow](https://img.shields.io/github/followers/CYB3RLEO?style=social)](https://github.com/CYB3RLEO)
