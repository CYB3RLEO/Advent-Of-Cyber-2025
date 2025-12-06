# 🎣 Advent of Cyber 2025 - Day 2: Phishing - Merry Clickmas

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-30%20min-blue?style=for-the-badge)](https://tryhackme.com)

## 📖 Room Overview

Join TBFC's red team to conduct an authorized phishing campaign testing employee security awareness. Learn to craft convincing phishing emails, deploy credential harvesters, and understand why humans remain the weakest link in cybersecurity.

### 🎯 Skills Learned

- Social engineering principles
- Phishing campaign development
- Social-Engineer Toolkit (SET)
- Credential harvesting
- Password reuse analysis
- Security awareness training

## 🔍 Challenge Summary

**Objective:** Test TBFC employees with authorized phishing simulation  
**Target:** factory@wareville.thm  
**Method:** Email spoofing + fake login page  
**Result:** Successful credential capture + password reuse discovery  

## 📝 Complete Walkthrough

### Step 1: Deploy Credential Harvester

Navigate to the challenge directory:
```bash
cd ~/Rooms/AoC2025/Day02
```

Start the fake login server:
```bash
./server.py
```

Output:
```
Starting server on http://0.0.0.0:8000
```

Test the page in Firefox: `http://127.0.0.1:8000`

### Step 2: Launch Social-Engineer Toolkit

Start SET:
```bash
setoolkit
```

Navigate through menus:
```
set> 1    # Social-Engineering Attacks
set> 5    # Mass Mailer Attack
set:mailer> 1    # E-Mail Attack Single Email Address
```

### Step 3: Configure Email Parameters

| Parameter | Value |
|-----------|-------|
| Send email to | `factory@wareville.thm` |
| Delivery method | `2` (Use own server) |
| From address | `updates@flyingdeer.thm` |
| From name | `Flying Deer` |
| SMTP server | `10.66.174.120` |
| Port | `25` |
| High priority | `no` |
| Attach file | `n` |
| Inline file | `n` |

### Step 4: Craft the Phishing Email

**Subject:**
```
Shipping Schedule Changes
```

**Body:**
```
Dear elves,

Kindly note that there have been significant changes to the shipping 
schedules due to increased shipping orders.

Please confirm the new schedule by visiting http://YOUR_IP:8000

Best regards,
Flying Deer
END
```

> Remember to type `END` (all capitals) on a new line to finish.

### Step 5: Capture Credentials

Wait 1-2 minutes and monitor the `server.py` terminal:

```
[CREDENTIAL CAPTURED]
Username: factory
Password: unranked-wisdom-anthem
```

### Step 6: Test Password Reuse

Navigate to: `http://10.66.174.120`

Login with captured credentials:
- Username: `factory`
- Password: `unranked-wisdom-anthem`

Check mailbox for delivery information.

## 🏆 Answers

**Question 1:** What is the password used to access the TBFC portal?  
**Answer:** `unranked-wisdom-anthem`

**Question 2:** What is the total number of toys expected for delivery?  
**Answer:** `1984000`

## 🎓 Key Concepts

### Social Engineering Psychology

Phishing exploits human psychology through:
- **Urgency**: Time pressure prevents critical thinking
- **Trust**: Impersonating familiar entities
- **Authority**: Appearing official or important
- **Curiosity**: Exploiting natural human inquisitiveness

### The S.T.O.P. Method (Detection)

**Before Acting:**
- **S**uspicious? Does something feel off?
- **T**elling me to click something?
- **O**ffering an amazing deal?
- **P**ushing me to act now?

**When Suspicious:**
- **S**low down - Scammers run on adrenaline
- **T**ype the address yourself
- **O**pen nothing unexpected
- **P**rove the sender (check real address)

### Types of Phishing

- **Email Phishing**: Traditional email-based attacks
- **Smishing**: SMS/text message phishing
- **Vishing**: Voice call phishing
- **Quishing**: QR code-based phishing
- **Social Media**: Direct message attacks

## 🛡️ Defense Strategies

### Technical Controls
```
✓ Multi-Factor Authentication (MFA)
✓ Email Authentication (SPF/DKIM/DMARC)
✓ Password Managers (unique passwords)
✓ Email Security Gateway
✓ Link Analysis/Sandboxing
✓ User Behavior Analytics
```

### Organizational Measures
```
✓ Regular phishing simulations
✓ Security awareness training
✓ Incident reporting procedures
✓ Password policies enforcement
✓ Security-first culture
```

## 📊 Attack Flow

```
┌─────────────┐
│  Attacker   │ 1. Deploys fake login page
│  (Red Team) │    python server.py :8000
└──────┬──────┘
       │
       │ 2. Crafts phishing email via SET
       │    From: updates@flyingdeer.thm
       │    Subject: Shipping Schedule Changes
       ▼
┌──────────────┐
│ Target User  │ 3. Receives convincing email
│   factory@   │    Clicks malicious link
│  wareville   │    Enters credentials
└──────┬───────┘
       │
       │ 4. Credentials captured
       │    factory:unranked-wisdom-anthem
       ▼
┌──────────────┐
│  Password    │ 5. Same password works on
│    Reuse     │    email portal
│  Discovered  │    Access to 1,984,000 records
└──────────────┘
```

## 🔧 Tools Reference

### Social-Engineer Toolkit Commands

| Action | Command |
|--------|---------|
| Launch SET | `setoolkit` |
| Social Engineering Menu | Option `1` |
| Mass Mailer | Option `5` |
| Single Email | Option `1` |
| Return to menu | Option `99` |
| Force exit | `Ctrl + C` |

### Python Server

```bash
# Start server
./server.py

# Test locally
curl http://127.0.0.1:8000

# Check from browser
firefox http://127.0.0.1:8000
```

## 📈 Real-World Statistics

- 📧 **90%** of data breaches begin with phishing
- 💰 Average phishing cost: **$4.65 million**
- 🎯 **74%** of organizations hit successfully
- 📊 **30%** of phishing emails get opened
- 🖱️ **12%** click malicious links
- 🚨 Only **3%** report suspicious emails

## ⚠️ Security Implications

### Why This Attack Succeeded

1. **Trust Exploitation**: Impersonated legitimate partner (Flying Deer)
2. **Plausible Context**: Shipping changes during busy season
3. **Clear Call-to-Action**: Simple request to confirm schedule
4. **No Technical Barriers**: No MFA or email authentication
5. **Password Reuse**: Same credentials across systems

### Red Flags Missed

- External link instead of internal portal
- Credential request via email link
- No prior communication about changes
- Sender domain not verified
- HTTP instead of HTTPS (no certificate)

## 💡 Lessons Learned

1. **Human Factor**: Training helps but doesn't eliminate risk
2. **Defense in Depth**: Multiple layers prevent single point of failure
3. **Password Hygiene**: Unique passwords limit breach scope
4. **MFA is Critical**: Would have prevented this completely
5. **Context Matters**: Attackers succeed by matching expectations

## 🔗 Related Resources

### TryHackMe Rooms
- [Phishing Prevention](https://tryhackme.com/room/phishing)
- [Social Engineering](https://tryhackme.com)
- [Red Team Fundamentals](https://tryhackme.com)

### External Resources
- [NIST Phishing Guide](https://www.nist.gov)
- [Anti-Phishing Working Group](https://apwg.org)
- [SET Documentation](https://github.com/trustedsec/social-engineer-toolkit)

## ⚖️ Legal Notice

**This walkthrough is for authorized security testing only.**

✅ **Permitted:**
- Authorized red team exercises
- Security awareness training
- Penetration testing with consent

❌ **Prohibited:**
- Unauthorized phishing campaigns
- Identity theft
- Unauthorized system access

**Unauthorized phishing is illegal and punishable by law. Always obtain written authorization.**

## 🤝 Contributing

Improvements welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## 📜 Disclaimer

Educational purposes only. All credit to TryHackMe and Advent of Cyber 2025 team. Author assumes no liability for misuse.

---

⭐ **Star this repo if it helped you!**

🎄 **Day 2/25 Complete! Stay vigilant!**
---

**Connect & Follow:**

[![GitHub](https://img.shields.io/github/followers/CYB3RLEO?style=social)](https://github.com/CYB3RLEO)
[![Twitter](https://img.shields.io/twitter/follow/_cyb3rleo?style=social)](https://x.com/_cyb3rleo)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=social&logo=linkedin)](https://linkedin.com/in/cyb3rle0)
