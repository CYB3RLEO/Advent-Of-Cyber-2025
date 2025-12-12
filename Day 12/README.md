# 🎄 TryHackMe Advent of Cyber 2025 - Day 12 🎄

## Phishmas Greetings: Master the Art of Phishing Detection

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/room/adventofcyber2025)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-30%20minutes-blue?style=for-the-badge)](https://tryhackme.com)
[![Day](https://img.shields.io/badge/Day-12-orange?style=for-the-badge)](https://tryhackme.com)

---

## 📖 Story Overview

Since McSkidy's disappearance, TBFC's defenses have crumbled. The Email Protection Platform is completely down, and with filters offline, the SOC staff must manually triage every suspicious message flooding their inboxes.

Malhare's Eggsploit Bunnies have seized this opportunity, launching a massive phishing campaign designed to steal credentials and disrupt SOC-mas preparations. Some emails disguise themselves as routine TBFC operations, volunteer sign-ups, or SOC-mas event logistics.

**Your Mission**: Join the Incident Response Task Force and identify which emails are legitimate and which are phishing attempts. Every wrong call brings Wareville closer to EAST-mas becoming reality! 🐰🥚

---

## 🎯 Learning Objectives

By completing this challenge, you'll master:

- ✅ **Phishing Email Identification** - Spot malicious messages like a pro
- ✅ **Trending Phishing Techniques** - Stay ahead of modern attack vectors
- ✅ **Spam vs. Phishing** - Understand the critical differences
- ✅ **Impersonation & Spoofing** - Recognize fake senders and domains
- ✅ **Social Engineering** - Identify emotional manipulation tactics
- ✅ **Typosquatting & Punycode** - Detect lookalike domains and Unicode tricks

---

## 🔍 Phishing vs. Spam: Know the Difference

### 📧 Spam: Digital Noise
- **Goal**: Marketing, promotion, traffic generation
- **Method**: Bulk distribution to anyone
- **Risk**: Mostly harmless annoyance
- **Example**: "Buy cheap watches now!"

### 🎣 Phishing: Precision Strike
- **Goal**: Credential theft, malware delivery, data exfiltration, financial fraud
- **Method**: Targeted, crafted messages with social engineering
- **Target**: Specific individuals or organizations
- **Risk**: HIGH THREAT - Can lead to complete compromise

**🔑 Key Difference**: **Intent**. Spam wants attention. Phishing wants access.

---

## 🛠️ Common Phishing Techniques

### 1. 👤 Impersonation

Attackers pretend to be someone trustworthy—a manager, IT department, or trusted service.

**🚩 Red Flags:**
- Free email domains (gmail.com, yahoo.com) for corporate communications
- Sender display name doesn't match actual email address
- Unusual email structure for the organization

**Example:**
```
From: McSkidy <mcskidy123@gmail.com>
Subject: URGENT: Reset your password immediately

❌ Red Flag: Corporate user with Gmail address
```

---

### 2. 🧠 Social Engineering

Manipulating emotions to trigger hasty, unthinking action.

**Common Emotional Triggers:**

| Emotion | Example Phrases | Purpose |
|---------|----------------|---------|
| **Urgency** | "URGENT", "IMMEDIATELY", "ACT NOW" | Force quick decisions |
| **Fear** | "Account will be closed", "Security breach detected" | Panic into action |
| **Authority** | "CEO requests", "IT Department mandates" | Exploit compliance |
| **Curiosity** | "You won't believe this...", "Secret document" | Trigger interest |
| **Helpfulness** | "Help needed urgently", "Colleague needs access" | Exploit good nature |

---

### 3. 📝 Typosquatting

Registering domains with common typos of legitimate sites.

**Examples:**

| Legitimate Domain | Typosquatted Version | Trick |
|-------------------|---------------------|--------|
| `github.com` | `glthub.com` | L vs I confusion |
| `tryhackme.com` | `tryhackrne.com` | RN looks like M |
| `google.com` | `gooogle.com` | Extra 'o' |
| `microsoft.com` | `microsoftonline.login444123.com` | Subdomain trickery |

**🛡️ Detection**: Carefully examine every character in domain names.

---

### 4. 🌐 Punycode (Homograph Attacks)

Using Unicode characters from different alphabets that look identical to Latin letters.

**How It Works:**
- Replace Latin `a` with Greek `α` (alpha)
- Use Cyrillic `с` instead of Latin `c`
- Browser displays lookalike, but actual domain is completely different

**Example:**
```
Display: tryhackme.com
Actual: тryhackme.com (Cyrillic т, г, у)
Encoded: xn--tryhackme-xyz123.com
```

**🛡️ Detection**: Check email headers for `xn--` prefix (ACE encoding) in the `Return-Path` field.

---

### 5. 📬 Email Spoofing

Making email appear to come from a legitimate domain while actually originating elsewhere.

**Email Authentication Checks:**

| Protocol | Purpose | What to Check |
|----------|---------|---------------|
| **SPF** | Specifies which servers can send email for domain | `SPF: pass` ✅ or `fail` ❌ |
| **DKIM** | Digital signature verification | `DKIM: pass` ✅ or `fail` ❌ |
| **DMARC** | Policy enforcement using SPF/DKIM | `DMARC: pass` ✅ or `fail` ❌ |

**🚩 Major Red Flags:**
```
From: mcskidy@tbfc.com
Return-Path: zxwsedr@easterbb.com

❌ Mismatch indicates spoofing!
```

---

### 6. 📎 Malicious Attachments

Classic phishing delivery method—still highly effective.

**High-Risk File Types:**

| File Type | Risk Level | Why Dangerous |
|-----------|-----------|---------------|
| `.html` / `.hta` | 🔴 CRITICAL | Runs scripts outside browser sandbox |
| `.exe` / `.scr` | 🔴 CRITICAL | Direct malware execution |
| `.pdf` | 🟡 MEDIUM | Can contain embedded scripts |
| `.docx` / `.xlsx` | 🟡 MEDIUM | Can execute macros |
| `.zip` / `.rar` | 🟡 MEDIUM | Often contains malware |

**Social Engineering Wrapper:**
```
Subject: Voice message from McSkidy
Attachment: voice_message_12_12_2025.html

❌ "Voice message" as HTML file? Major red flag!
```

---

## 🔥 Trending Phishing Techniques (2025)

### 1. ☁️ Legitimate Service Abuse

Attackers hide behind trusted platforms to bypass email filters:

**Common Platforms Exploited:**
- 📦 Dropbox shared links
- 📄 Google Drive documents
- 💼 OneDrive files
- 📨 WeTransfer downloads

**Why This Works:**
- ✅ Domains are trusted (dropbox.com, drive.google.com)
- ✅ Passes most email security filters
- ✅ Users inherently trust these platforms

**Attack Flow:**
```
1. Phishing email with legitimate service link
   ↓
2. Shared document with attractive content
   ↓
3. Link to fake login page OR malicious download
   ↓
4. Credentials stolen / Malware installed
```

---

### 2. 🔐 Fake Login Pages

High-fidelity replicas of legitimate login portals.

**Common Targets:**
- Microsoft Office 365 / Outlook
- Google Workspace / Gmail
- Corporate VPN gateways
- Email webmail portals

**🛡️ Detection Checklist:**
- [ ] Examine URL character-by-character
- [ ] Look for typos: `microsoftonline.login444123.com`
- [ ] Verify HTTPS and valid SSL certificate
- [ ] Check certificate issuer and domain ownership
- [ ] Bookmark legitimate login pages and use only those

---

### 3. 💬 Side-Channel Communication

Moving conversations off monitored email to unprotected channels.

**Methods Used:**
- 📱 SMS/Text messages
- 💬 WhatsApp, Telegram, Signal
- ☎️ Phone calls to personal numbers
- 📧 Personal email accounts

**Purpose**: Continue social engineering without security team monitoring.

**🚩 Red Flag Examples:**
- "Contact me on WhatsApp at +123-456-7890"
- "Call this number instead of my office line"
- "Let's continue this conversation via text"

---

## 🧪 Practical Email Analysis Walkthrough

**Access the lab**: `https://LAB_WEB_URL.p.thmlabs.com`

### 📋 Email Classification Framework

For each email, systematically identify:

1. **Type**: Phishing / Spam / Legitimate
2. **Techniques** (if phishing): Select 3 specific indicators
3. **Evidence**: Document red flags found

---

### ✅ Analysis Checklist

#### **Sender Analysis**
- [ ] Domain matches expected organization
- [ ] Email structure follows company standards
- [ ] No free email domains (gmail, yahoo, hotmail)
- [ ] No punycode characters (`xn--` in headers)
- [ ] Display name matches actual email address

#### **Header Analysis**
- [ ] `SPF: pass` (Sender Policy Framework)
- [ ] `DKIM: pass` (DomainKeys Identified Mail)
- [ ] `DMARC: pass` (Domain-based Message Authentication)
- [ ] `Return-Path` matches `From` address

#### **Content Analysis**
- [ ] No artificial urgency language
- [ ] Professional, appropriate tone
- [ ] No grammar or spelling errors
- [ ] Reasonable, context-appropriate request
- [ ] No suspicious or shortened links
- [ ] No unusual or unexpected attachments

#### **Context Analysis**
- [ ] Expected communication for this sender
- [ ] Sender is known to recipient
- [ ] Request makes logical sense
- [ ] No side-channel communication requests

---

## 🚩 Challenge Solutions & Flags

### Email 1
**Classification**: 🎣 **Phishing**  
**Flag**: `THM{yougotnumber1-keep-it-going}`

**Key Indicators:**
- Suspicious sender domain
- Urgency in subject line
- Unusual attachment type

---

### Email 2
**Classification**: 🎣 **Phishing**  
**Flag**: `THM{nmumber2-was-not-tha-thard!}`

**Key Indicators:**
- Failed SPF/DKIM authentication
- Return-Path mismatch
- Typosquatted domain

---

### Email 3
**Classification**: 🎣 **Phishing**  
**Flag**: `THM{Impersonation-is-areal-thing-keepIt}`

**Key Indicators:**
- Impersonation attempt
- Free email domain for corporate user
- Social engineering tactics

---

### Email 4
**Classification**: 🎣 **Phishing**  
**Flag**: `THM{Get-back-SOC-mas!!}`

**Key Indicators:**
- Punycode/homograph attack
- `xn--` in Return-Path header
- Lookalike domain

---

### Email 5
**Classification**: 📧 **Spam** (NOT Phishing!)  
**Flag**: `THM{It-was-just-a-sp4m!!}`

**Key Differentiator:**
- Marketing content, not credential theft
- Bulk distribution, not targeted attack
- Annoying but not malicious intent

⚠️ **Important**: Email 5 demonstrates the critical difference between spam and phishing!

---

### Email 6
**Classification**: 🎣 **Phishing**  
**Flag**: `THM{number6-is-the-last-one!-DX!}`

**Key Indicators:**
- Malicious attachment
- Legitimate service abuse
- Social engineering wrapper

---

## 📊 Real-World Impact

### Statistics That Matter

| Metric | Value | Source |
|--------|-------|--------|
| Attacks starting with phishing | **90%+** | Verizon DBIR 2024 |
| Phishing emails sent | **1 in 99** | Proofpoint |
| Average incident cost | **$4.65M** | IBM Cost of Data Breach |
| Organizations attacked in 2024 | **74%** | APWG |

### Notable Incidents

**Google/Facebook (2013-2015)**
- 💰 **Loss**: $100M stolen
- 🎯 **Method**: Business Email Compromise (BEC)
- 📧 **Vector**: Fake invoices from spoofed suppliers

**FACC (2016)**
- 💰 **Loss**: €50M
- 🎯 **Method**: CEO fraud
- 📧 **Vector**: Impersonation of CEO

**Ubiquiti Networks (2015)**
- 💰 **Loss**: $46.7M
- 🎯 **Method**: BEC with urgency
- 📧 **Vector**: Fake employee requests

---

## 🛡️ Defense Strategies

### For Users

| Action | Why It Matters |
|--------|---------------|
| ✅ **Verify sender before acting** | Prevents impersonation success |
| ✅ **Hover over links** | Reveals true destination URL |
| ✅ **Check email headers** | Exposes spoofing attempts |
| ✅ **Report phishing** | Helps protect others |
| ✅ **Never share credentials via email** | Legitimate companies never ask |
| ✅ **When in doubt, verify via known channels** | Use bookmarked sites or phone |

### For Organizations

| Control | Purpose |
|---------|---------|
| ✅ **Email authentication (SPF/DKIM/DMARC)** | Prevent domain spoofing |
| ✅ **Advanced email filtering** | Block known phishing patterns |
| ✅ **Security awareness training** | Educate users on threats |
| ✅ **Phishing simulations** | Test user readiness |
| ✅ **Multi-factor authentication** | Protect even if credentials stolen |
| ✅ **Incident response procedures** | Quick containment when breached |

---

## 🎓 Key Takeaways

1. **🎯 Phishing is the #1 Initial Access Vector** - Despite advanced technology, human psychology remains the weakest link.

2. **🧠 Intent Matters** - Spam is annoying; phishing is dangerous. Know the difference.

3. **🔍 Details Matter** - One misplaced character in a domain can mean the difference between safety and compromise.

4. **🚨 Urgency is a Red Flag** - Legitimate communications rarely demand immediate action without proper verification.

5. **🛡️ Defense is Layered** - Technology helps, but human vigilance is essential.

6. **📚 Continuous Learning** - Attackers evolve constantly. Your knowledge must too.

---

## 🔗 Essential Commands Reference

### Email Header Analysis

```bash
# View email source/headers in most email clients
Ctrl + U (Gmail web)
Ctrl + Alt + H (Outlook)
View > Message Source (Thunderbird)

# Extract specific header fields
grep -i "Return-Path" email.eml
grep -i "SPF" email.eml
grep -i "DKIM" email.eml
grep -i "DMARC" email.eml

# Check for punycode encoding
grep -i "xn--" email.eml
```

### URL Analysis

```bash
# Decode URL-encoded strings
python3 -c "import urllib.parse; print(urllib.parse.unquote('encoded_url'))"

# Check URL reputation (VirusTotal API)
curl -s "https://www.virustotal.com/api/v3/urls/URL_ID" \
  -H "x-apikey: YOUR_API_KEY"

# Analyze shortened URLs without clicking
curl -I -L short.url.link
```

### Domain Investigation

```bash
# Check domain registration
whois suspicious-domain.com

# DNS lookup
nslookup suspicious-domain.com
dig suspicious-domain.com

# Check SPF records
dig TXT suspicious-domain.com | grep spf

# Check DMARC records
dig TXT _dmarc.suspicious-domain.com
```

---

## 🎯 Related TryHackMe Rooms

Continue your security awareness journey:

| Room | Focus Area | Difficulty |
|------|-----------|-----------|
| **Phishing Analysis Fundamentals** | Email forensics deep dive | Easy |
| **Phishing Prevention** | Building organizational defenses | Medium |
| **Email Security** | SPF, DKIM, DMARC configuration | Medium |
| **Social Engineering** | Complete manipulation tactics | Medium |

---

## 📚 Additional Resources

### Documentation & Learning
- [OWASP Phishing Guide](https://owasp.org/www-community/attacks/Phishing)
- [Anti-Phishing Working Group (APWG)](https://apwg.org/)
- [Phishing.org Resources](https://www.phishing.org/)

### Tools
- [PhishTool](https://www.phishtool.com/) - Email analysis platform
- [VirusTotal](https://www.virustotal.com/) - URL & file reputation
- [URLScan.io](https://urlscan.io/) - Website scanner
- [MXToolbox](https://mxtoolbox.com/) - Email header analyzer

---

## 🤝 Contributing

Found an error or have suggestions? Contributions are welcome!

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

---

## 📜 License

This walkthrough is provided for educational purposes as part of TryHackMe's Advent of Cyber 2025 challenge.

**Disclaimer**: All techniques described should only be used in authorized security testing environments.

---

## 🌟 Support

If this walkthrough helped you:
- ⭐ Star this repository
- 🔄 Share with fellow security enthusiasts
- 💬 Connect with me on [LinkedIn](#) | [Twitter](#)

---


<div align="center">

### 🎄 Happy Phishing Hunting! Stay Safe! 🎄

**Think Before You Click. Verify Before You Act.**

Made with ❤️ for the cybersecurity community

![TryHackMe](https://tryhackme.com/img/logo/tryhackme_logo_full.svg)

</div>

---

## Tags

`#TryHackMe` `#AdventOfCyber2025` `#Phishing` `#EmailSecurity` `#SocialEngineering` `#CyberSecurity` `#SecurityAwareness` `#ThreatDetection` `#DFIR` `#BlueTeam` `#SOC` `#InfoSec` `#CyberDefense`
