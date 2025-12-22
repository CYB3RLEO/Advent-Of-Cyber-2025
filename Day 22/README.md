# 🎄 TryHackMe Advent of Cyber 2025 - Day 22: C2 Detection - Command & Carol

![TryHackMe](https://img.shields.io/badge/TryHackMe-AoC%202025-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Duration](https://img.shields.io/badge/Duration-60%20Minutes-blue?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-C2%20Detection-darkblue?style=for-the-badge)
![Tool](https://img.shields.io/badge/Tool-RITA-green?style=for-the-badge)

---

## 🎯 Overview

**Room**: Advent of Cyber 2025 - Day 22  
**Title**: C2 Detection - Command & Carol  
**Topic**: Command and Control Detection using RITA  
**Difficulty**: Medium  
**Duration**: 60 minutes  
**Tools**: RITA, Zeek, PCAP analysis

Learn to detect Command and Control (C2) communication by converting PCAP files to Zeek logs and analyzing them with RITA (Real Intelligence Threat Analytics). Identify beaconing behavior, DNS tunneling, data exfiltration, and malicious infrastructure.

---

## 📖 The Story

### 🔍 The Hunt Begins

The TBFC is extremely wary after King Malhare's recent attacks. They're on full alert, but it's **too quiet**.

**Sir Elfo takes initiative**: "Let's hunt for C2 traffic using our collected network captures!"

**TBFC elves object**: "We don't have time to analyze all that traffic!"

**Sir Elfo chuckles**: "Don't fret! I have **RITA** — Real Intelligence Threat Analytics. Just convert PCAPs to Zeek logs, and RITA does the rest!"

**Your mission**: Hunt down the "malrabbits" by analyzing network traffic for C2 communication patterns.

---

## 🎓 Learning Objectives

By completing this challenge, you will:

- ✅ Convert **PCAP files** to **Zeek logs**
- ✅ Use **RITA** to analyze network traffic
- ✅ Detect **C2 beacons**, **DNS tunneling**, and **data exfiltration**
- ✅ Understand **threat modifiers** and **severity scoring**
- ✅ Master RITA's **search functionality**
- ✅ Integrate **threat intelligence feeds**
- ✅ Identify connection patterns indicating compromise

---

## 🔬 Understanding RITA

### What is RITA?

**RITA (Real Intelligence Threat Analytics)** = Open-source C2 detection framework by Active Countermeasures

### Core Features

| Feature | Purpose |
|---------|---------|
| **C2 Beacon Detection** | Identifies periodic communication patterns |
| **DNS Tunneling Detection** | Spots data exfiltration via DNS |
| **Long Connection Detection** | Flags persistent connections |
| **Data Exfiltration Detection** | Identifies large outbound transfers |
| **Threat Intel Integration** | Cross-references known malicious IPs |
| **Severity Scoring** | Ranks connections by threat level |
| **Host Tracking** | Shows internal → external communication |
| **First Seen Analysis** | Tracks when external hosts appeared |

### How RITA Works

**Correlates fields**:
- IP addresses
- Ports
- Timestamps
- Connection durations
- DNS queries
- Certificate details

**Analyzes for**:
- ⏱️ Periodic connection intervals
- 🌐 Excessive DNS queries
- 📏 Long FQDNs
- 🎲 Random subdomains
- 📊 Data volume over time
- 🔒 Self-signed/short-lived certificates
- 🚩 Known malicious IPs

---

## 🦓 Understanding Zeek

### What is Zeek?

**Zeek** (formerly Bro) = Open-source Network Security Monitoring (NSM) tool

**What it's NOT**:
- ❌ Firewall
- ❌ IPS/IDS
- ❌ Signature-based blocker

**What it IS**:
- ✅ Passive traffic observer
- ✅ Protocol analyzer
- ✅ Structured log generator
- ✅ Content extractor

### Input Methods

```
SPAN ports → Zeek
Network taps → Zeek
PCAP files → Zeek
```

### Output: Structured Logs

| Log Type | Content |
|----------|---------|
| `conn.log` | Connection summaries |
| `dns.log` | DNS queries/responses |
| `http.log` | HTTP traffic |
| `ssl.log` | TLS/SSL certificates |
| `files.log` | Transferred files |
| `x509.log` | Certificate details |

**Critical**: RITA only accepts Zeek logs, not raw PCAPs!

---

## 🚀 Complete Walkthrough

### Phase 1: Environment Exploration

```bash
cd ~
ls
```

**Output**:
```
pcaps/        # Example PCAPs (Bradley Duncan's real malware samples)
zeek_logs/    # Converted Zeek logs
```

---

### Phase 2: Convert PCAP to Zeek Logs

#### Example Conversion

```bash
zeek readpcap pcaps/AsyncRAT.pcap zeek_logs/asyncrat
```

**Output**:
```
Starting the Zeek docker container
Zeek logs will be saved to /home/ubuntu/zeek_logs/asyncrat
```

#### Examine Generated Logs

```bash
cd zeek_logs/asyncrat/
ls
```

**Zeek Logs Created**:
```
conn.log       # Connections
dns.log        # DNS queries
http.log       # HTTP traffic
ssl.log        # SSL/TLS
files.log      # File transfers
x509.log       # Certificates
[+ more logs]
```

---

### Phase 3: Import into RITA

```bash
rita import --logs ~/zeek_logs/asyncrat/ --database asyncrat
```

**Processing**:
```
[THREAT INTEL] Updating online feed...
[-] Parsing: conn.log
[-] Parsing: http.log
[-] Parsing: ssl.log
[-] Parsing: dns.log
Log Parsing ████████████ 4/4
```

**Note**: Larger datasets = better insights, fewer false positives

---

### Phase 4: View Results

```bash
rita view asyncrat
```

**Opens terminal GUI** with three sections:
1. Search Bar
2. Results Pane
3. Details Pane

---

## 🖥️ RITA Interface Guide

### 1. Search Bar

**Activate**: Press `/`

**Search Fields**:
```bash
dst:example.com          # Destination filter
src:10.0.0.5            # Source IP filter
beacon:>=70             # Beacon score ≥ 70%
severity:>=8            # Severity ≥ 8
sort:duration-desc      # Sort by duration (descending)
```

**Combined Search**:
```bash
dst:rabbithole.malhare.net beacon:>=70 sort:duration-desc
```

**Help**: Press `?` in search mode  
**Exit**: Press `Esc`

---

### 2. Results Pane

| Column | Description | Threat Indicator |
|--------|-------------|------------------|
| **Severity** | Threat score | Higher = more suspicious |
| **Source** | Internal host | Potentially compromised |
| **Destination** | External host | Potential C2 server |
| **Beacon %** | C2 likelihood | High % = likely C2 |
| **Duration** | Connection length | Long = suspicious |
| **Subdomains** | Unique subdomains | Many = C2/exfiltration |
| **Threat Intel** | Known IOC match | Hit = confirmed malicious |

---

### 3. Details Pane

#### A. Threat Modifiers

| Modifier | Meaning | Why Suspicious |
|----------|---------|----------------|
| **MIME/URI Mismatch** | Header doesn't match URI | Evasion technique |
| **Rare Signature** | Unusual patterns | Attacker oversight |
| **Prevalence** | % of hosts contacting destination | Low % = not widely used |
| **First Seen** | When first observed | New = more suspicious |
| **Missing Host Header** | No HTTP host header | Misconfiguration/malware |
| **Large Outbound Data** | High volume sent | Data exfiltration |
| **No Direct Connections** | Proxied/tunneled | Complex C2 |

#### B. Connection Info

| Field | Description | Indicator |
|-------|-------------|-----------|
| **Connection Count** | # of connections | Very high = beacon |
| **Total Bytes Sent** | Data volume | High = exfiltration |
| **Port-Protocol-Service** | Connection details | Non-standard = suspicious |

---

## 🏁 Challenge Solution

### Objective

Analyze `~/pcaps/rita_challenge.pcap` for King Malhare's C2 infrastructure.

### Step 1: Convert PCAP

```bash
zeek readpcap ~/pcaps/rita_challenge.pcap ~/zeek_logs/rita_challenge
```

### Step 2: Import to RITA

```bash
rita import --logs ~/zeek_logs/rita_challenge/ --database rita_challenge
```

### Step 3: View Results

```bash
rita view rita_challenge
```

### Step 4: Answer Questions

#### Q1: Hosts Communicating with malhare.net

**Search**: `/dst:malhare.net`

**Look for**: Prevalence modifier

**Method**: The Prevalence threat modifier shows the number of internal hosts communicating with the destination.

**Answer**: `6`

---

#### Q2: Threat Modifier for Host Count

**Question**: Which modifier indicates host count?

**Concept**: Prevalence analyzes how many internal hosts contact a specific external host.

**Answer**: `prevalence`

---

#### Q3: Highest Connection Count

**Search**: `/dst:rabbithole.malhare.net`

**Look for**: Connection Count in Connection Info

**Method**: Navigate entries with arrow keys, find highest count

**Answer**: `40`

---

#### Q4: Complex Search Filter

**Requirements**:
- Destination: `rabbithole.malhare.net`
- Beacon: `≥ 70%`
- Sort: Connection duration (descending)

**Search Syntax**:
```bash
dst:rabbithole.malhare.net beacon:>=70 sort:duration-desc
```

**Components**:
- `dst:` = Destination filter
- `beacon:>=70` = Beacon score ≥ 70
- `sort:duration-desc` = Sort by duration (↓)

**Answer**: `dst:rabbithole.malhare.net beacon:>=70 sort:duration-desc`

---

#### Q5: Port Used by Host

**Search**: `/dst:rabbithole.malhare.net src:10.0.0.13`

**Look for**: Port in Connection Info

**Method**: Select matching entry, examine Port-Protocol-Service

**Answer**: `80`

---

## 🎯 Flags & Answers

| Question | Answer |
|----------|--------|
| How many hosts are communicating with malhare.net? | `6` |
| Which Threat Modifier tells us the number of hosts? | `prevalence` |
| Highest number of connections to rabbithole.malhare.net? | `40` |
| Complex search filter? | `dst:rabbithole.malhare.net beacon:>=70 sort:duration-desc` |
| Which port did 10.0.0.13 use? | `80` |

---

## 🎯 C2 Detection Patterns

### 1. Beacon Behavior

**Characteristics**:
```
Periodic communication (every X seconds/minutes)
Consistent intervals
Regular check-ins
```

**Example**:
```
10:00:00 - Connection 1
10:05:00 - Connection 2 (+5 min)
10:10:00 - Connection 3 (+5 min)
10:15:00 - Connection 4 (+5 min)
Pattern: Consistent 5-minute beacon
```

**RITA Detection**:
- Analyzes timestamps
- Calculates intervals
- Scores beacon likelihood

---

### 2. DNS Tunneling

**Technique**: Exfiltrate data via DNS queries

**Indicators**:
- Excessive queries
- Very long FQDNs
- Random subdomains
- Unusual TXT records

**Example**:
```
data1.exfil.attacker.com
data2.exfil.attacker.com
ZGF0YQ==.exfil.attacker.com (Base64 in subdomain)
```

---

### 3. Long Connections

**Normal**: Most protocols close quickly

**Legitimate Long Connections**:
- SSH sessions
- RDP connections
- VNC sessions

**Malicious Long Connections**:
- C2 channels
- Data exfiltration
- Interactive backdoors

---

### 4. Data Exfiltration

**Indicators**:
- Large outbound volumes
- Unusual destinations
- Non-standard ports
- Encrypted channels

**RITA Tracking**:
- Measures bytes sent
- Compares to baseline
- Flags anomalies

---

## 📚 Command Reference

### Zeek Commands

```bash
# Convert PCAP to Zeek logs
zeek readpcap <pcap_file> <output_dir>

# Example
zeek readpcap pcaps/AsyncRAT.pcap zeek_logs/asyncrat

# View Zeek log
cat zeek_logs/asyncrat/conn.log
```

### RITA Commands

```bash
# Import Zeek logs
rita import --logs <zeek_dir> --database <db_name>

# View results
rita view <db_name>

# List databases
rita list

# Delete database
rita delete <db_name>

# Show RITA version
rita --version
```

### RITA Search Syntax

```bash
# Press / to activate search

# Basic filters
dst:example.com           # Destination
src:10.0.0.5             # Source
beacon:>=70              # Beacon score
severity:>=8             # Severity

# Sorting
sort:severity-desc       # By severity (↓)
sort:duration-desc       # By duration (↓)
sort:connections-desc    # By connections (↓)

# Combined
dst:malhare.net beacon:>=70 sort:connections-desc
```

---

## 🛡️ Defense Workflow

### 1. Collection

```
Network taps/SPAN ports
    ↓
Zeek monitoring
    ↓
PCAP storage
```

### 2. Analysis

```
PCAP files
    ↓
Convert to Zeek logs
    ↓
Import to RITA
    ↓
Analyze results
```

### 3. Investigation

```
High-severity entries
    ↓
Threat intel hits
    ↓
Connection patterns
    ↓
Pivot to detailed logs
    ↓
Confirm IOCs
```

### 4. Response

```
Block malicious IPs
    ↓
Isolate compromised hosts
    ↓
Remove malware
    ↓
Update detection rules
```

---

## 💡 Key Takeaways

✅ **RITA** automates C2 detection through pattern analysis

✅ **Zeek logs** provide structured network data for analysis

✅ **Threat modifiers** prioritize investigations by severity

✅ **Beacon detection** identifies periodic C2 patterns

✅ **DNS tunneling** spotted via excessive queries/long FQDNs

✅ **Long connections** suspicious for most protocols

✅ **Prevalence** shows how many hosts contact destinations

✅ **Threat intel** provides instant IOC identification

✅ **Search filters** enable targeted investigation

✅ **Large datasets** improve accuracy, reduce false positives

---

## 🔗 Related Rooms

Continue your network security journey:

- **Zeek Exercises**: Manual Zeek log analysis
- **Wireshark Basics**: Deep PCAP analysis
- **Network Forensics**: Advanced traffic analysis
- **Threat Hunting**: Proactive security investigation
- **Threat Intelligence**: IOC integration and usage

---

## 🤝 Contributing

Improvements welcome!

1. Fork repository
2. Create feature branch
3. Commit changes
4. Push and open Pull Request

---

## 📄 License

Educational purposes - TryHackMe Advent of Cyber 2025

---

## 🙏 Acknowledgments

- **TryHackMe** for Advent of Cyber 2025
- **Active Countermeasures** for RITA
- **Zeek Project** for network monitoring tools
- **Bradley Duncan** for malware PCAP samples

---

## 🎯 Analysis Tips

### Effective C2 Detection

**Start with**:
1. Threat intel hits (known malicious)
2. High severity scores
3. High beacon percentages
4. Unusual connection counts

**Look for combinations**:
- Beacon + long duration + non-standard port
- DNS tunneling + large subdomains + excessive queries
- Data exfiltration + high bytes sent + rare destination

### False Positive Reduction

**Consider**:
- Dataset size (larger = more accurate)
- Legitimate services (CDNs, cloud providers)
- Internal applications (proxies, monitoring tools)
- Business context (VPNs, remote access)

---

**🎄 Happy Hunting! Stay Vigilant! 🔍**

*Proactive threat hunting finds compromises before they become incidents.*
