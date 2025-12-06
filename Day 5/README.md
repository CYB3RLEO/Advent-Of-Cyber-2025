# 🎄 Advent of Cyber 2025 - Day 5: IDOR - Santa's Little IDOR

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)
[![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-45%20min-blue?style=for-the-badge)](https://tryhackme.com)
[![OWASP](https://img.shields.io/badge/OWASP-Top%2010-critical?style=for-the-badge)](https://owasp.org)

## 📖 Overview

Day 5 explores Insecure Direct Object Reference (IDOR), one of the most common web application vulnerabilities. Investigate the TryPresentMe website to discover how Sir Carrotbane exploited IDOR to steal vouchers and conduct targeted phishing attacks.

### 🎯 Learning Objectives

- Understand authentication vs. authorization concepts
- Identify IDOR opportunities in web applications
- Exploit various IDOR implementations (numeric, encoded, hashed, UUID)
- Learn horizontal privilege escalation techniques
- Implement secure remediation strategies

## 🔍 Investigation Summary

**Threat Actor:** Sir Carrotbane (HopSec Island)  
**Target:** TryPresentMe website  
**Vulnerability:** IDOR across multiple endpoints  
**Impact:** Voucher theft + data exfiltration for phishing  
**Attack Type:** Horizontal privilege escalation

## 🧪 Lab Setup

**Target URL:** `http://10.65.177.232`  
**Credentials:**
- Username: `niels`
- Password: `TryHackMe#2025`

## 📚 IDOR Fundamentals

### What is IDOR?

**Insecure Direct Object Reference** occurs when:
1. Application uses direct references (IDs) to access objects
2. No authorization check before returning data
3. Attacker manipulates references to access unauthorized resources

### Authentication vs. Authorization

| Concept | Definition | Question Answered |
|---------|------------|-------------------|
| **Authentication** | Verifying identity | "Who are you?" |
| **Authorization** | Verifying permissions | "What can you access?" |

**Critical Rule:** Authorization cannot happen before authentication.

### Privilege Escalation Types

**Vertical**: Gaining higher privileges (user → admin)  
**Horizontal**: Accessing peer-level data (user A → user B's data)

**IDOR = Typically horizontal privilege escalation**

## 📝 Complete Walkthrough

### Challenge 1: Simple Numeric IDOR 🔢

**Steps:**
1. Login with provided credentials
2. Open Developer Tools (F12) → Network tab
3. Refresh page and observe `view_accountinfo` request
4. Note the `user_id=10` parameter

**Exploitation:**
1. Go to Storage tab → Local Storage
2. Find `auth_user` entry
3. Change `user_id` from `10` to `11`
4. Refresh page
5. You're now viewing user 11's data!

**Testing for 10 Children:**
```
Iterate through user_ids:
user_id=10 → count children
user_id=11 → count children
user_id=12 → count children
...
user_id=15 → FOUND: 10 children!
```

**🚩 Answers:**
- **IDOR stands for?** `Insecure Direct Object Reference`
- **Privilege escalation type?** `Horizontal`
- **Parent with 10 children?** `15`

### Challenge 2: Base64 Encoded References 🔐

**Observation:**
- Click eye icon next to child profile
- Request: `/api/child/Mg==`

**Decoding:**
```bash
echo "Mg==" | base64 -d
# Output: 2
```

**Exploitation:**
```bash
# Encode different numbers
echo -n "1" | base64  # MQ==
echo -n "3" | base64  # Mw==
echo -n "4" | base64  # NA==
```

**Key Insight:** Base64 is encoding, not encryption. No security provided.

### Challenge 3: MD5 Hashed References 🔨

**Observation:**
- Click edit icon next to child
- URL contains MD5 hash: `c4ca4238a0b923820dcc509a6f75849b`

**Exploitation:**
```bash
# Generate MD5 hashes
echo -n "1" | md5sum  # c4ca4238a0b923820dcc509a6f75849b
echo -n "2" | md5sum  # c81e728d9d4c2f636f067f89cc14862c
echo -n "3" | md5sum  # eccbc87e4b5ce2fe28308fd9f2a7baf3
```

**Attack Method:**
1. Generate MD5 hashes for sequential numbers
2. Test each hash against the endpoint
3. Access unauthorized child records

**Key Insight:** Hashing predictable values doesn't prevent IDOR.

### Challenge 4: UUID Version 1 Vulnerability ⏰

**Observation:**
- Vouchers use UUID format
- Example: `550e8400-e29b-41d4-a716-446655440000`

**Investigation:**
Use UUID decoder tool to analyze vouchers.

**Vulnerability:**
- UUID v1 includes timestamp
- Vouchers generated between 20:00-24:00 UTC
- Can generate all possible UUIDs for that timeframe
- 3,600 - 14,400 possible UUIDs depending on range

**Attack Concept:**
```python
import uuid
import datetime

base_time = datetime.datetime(2025, 11, 20, 20, 0, 0)
for seconds in range(14400):  # 4 hours
    timestamp = base_time + datetime.timedelta(seconds=seconds)
    # Generate UUID v1 for this timestamp
    # Test against /parents/vouchers/claim
```

**Key Insight:** UUID v1 is NOT secure for secrets. Use UUID v4 (random).

## 🛡️ Prevention & Remediation

### ❌ Vulnerable Code

```python
@app.route('/api/child/<child_id>')
def get_child(child_id):
    child = db.query("SELECT * FROM children WHERE id = ?", child_id)
    return jsonify(child)
```

### ✅ Secure Code

```python
@app.route('/api/child/<child_id>')
def get_child(child_id):
    user_id = session['user_id']
    # Check authorization
    child = db.query(
        "SELECT * FROM children WHERE id = ? AND parent_id = ?",
        child_id, user_id
    )
    if not child:
        return jsonify({"error": "Unauthorized"}), 403
    return jsonify(child)
```

## 🔒 Security Best Practices

### 1. Server-Side Authorization ✅

**Always verify:**
- Who is making the request?
- Are they authorized for this specific resource?
- Check on EVERY request, not just login

### 2. Indirect References

```python
# Instead of direct IDs
/api/account/12345

# Use contextual references
/api/account/me
# Server maps 'me' to authenticated user's ID
```

### 3. Unpredictable Identifiers

**Good:**
- UUID v4 (random)
- Cryptographically secure random strings
- Non-sequential tokens

**Bad:**
- Sequential integers
- Timestamp-based UUIDs (v1)
- Encoded sequential numbers

**Remember:** Random IDs alone ≠ Security. Authorization still required!

### 4. Monitoring & Logging

```
[ALERT] User 10 attempted access to user 11's data - DENIED
[ALERT] User 10 attempted access to user 12's data - DENIED
[ALERT] User 10 attempted access to user 13's data - DENIED
[WARNING] Potential IDOR attack from User 10
```

### 5. Testing Methodology

**Manual Test:**
1. Login as User A
2. Note resource ID (e.g., `/api/data/123`)
3. Login as User B
4. Try accessing User A's resource (`/api/data/123`)
5. Should be DENIED

**Automated Test:**
- Use Burp Suite Intruder
- Test all endpoints with different user contexts
- Verify 403 Forbidden responses

## 🏆 Challenge Answers

| Question | Answer |
|----------|--------|
| What does IDOR stand for? | `Insecure Direct Object Reference` |
| What type of privilege escalation? | `Horizontal` |
| Parent with 10 children (user_id)? | `15` |

## 💡 Bonus Challenges

### Bonus 1: Finding Specific Child

**Objective:** Find `id_number` of child born on 2019-04-17

**Method:**
1. Use Burp Suite Intruder
2. Target Base64 or MD5 endpoint
3. Generate payloads (sequential numbers)
4. Filter responses for birthdate: 2019-04-17
5. Extract id_number

### Bonus 2: Valid Voucher Discovery

**Objective:** Find voucher valid on November 20, 2025 (20:00-24:00 UTC)

**Method:**
1. Generate UUID v1 for every second in timeframe
2. Test against `/parents/vouchers/claim`
3. Identify successful claim

## 🎓 Key Takeaways

### The IDOR Formula

```
IDOR = Direct Reference + Missing Authorization
SDOR = Direct Reference + Authorization Check ✅
```

### Security Principles

1. **Never trust user input** - All IDs can be manipulated
2. **Authentication ≠ Authorization** - Separate concerns
3. **Obscurity ≠ Security** - Encoding/hashing doesn't protect
4. **Defense in depth** - Multiple controls required

### IDOR Testing Checklist

- [ ] Identify all direct references in URLs/APIs
- [ ] Test with different user accounts
- [ ] Try sequential ID manipulation
- [ ] Check for encoded references (Base64, hex)
- [ ] Look for hashed references (MD5, SHA1)
- [ ] Examine UUID patterns (v1 vs v4)
- [ ] Test all HTTP methods (GET, POST, PUT, DELETE)
- [ ] Verify authorization on all CRUD operations

## 🌐 Real-World Impact

### Notable IDOR Vulnerabilities

**Facebook (2013)**
- Could view private photos via ID manipulation
- Impact: Millions of users affected

**PayPal (2014)**
- Could download other users' invoices
- Impact: Financial data exposure

**Instagram (2015)**
- Could delete any user's photos
- Impact: Data integrity compromise

## 🔗 Related Resources

### TryHackMe Rooms
- [Complete IDOR Room](https://tryhackme.com)
- [OWASP Top 10](https://tryhackme.com)
- [Web Application Security](https://tryhackme.com)

### External Resources
- [OWASP Testing Guide - IDOR](https://owasp.org/www-project-web-security-testing-guide/)
- [PortSwigger IDOR Labs](https://portswigger.net/web-security/access-control)
- [HackerOne IDOR Reports](https://hackerone.com/reports)

## 🛠️ Tools for IDOR Testing

**Manual Testing:**
- Browser Developer Tools
- Burp Suite Community/Pro
- OWASP ZAP

**Automation:**
- Autorize (Burp Extension)
- AuthMatrix (Burp Extension)
- Custom Python scripts

## 📊 Attack Patterns

### Pattern Recognition

| Reference Type | Example | Exploitation |
|---------------|---------|--------------|
| Numeric | `id=123` | Increment/decrement |
| Base64 | `id=MTIz` | Decode, modify, encode |
| MD5 | `id=202cb...` | Hash sequential values |
| UUID v1 | `550e8400-...` | Generate by timestamp |
| UUID v4 | `f47ac10b-...` | No pattern (secure) |

## 🤝 Contributing

Found an issue or improvement? Open a PR!

## 📜 License

Educational purposes only. All credit to TryHackMe and Advent of Cyber 2025.

---

⭐ **Found this helpful? Star the repo!**

🔒 **Day 5/25 Complete - Always check authorization!**

[![Follow](https://img.shields.io/github/followers/CYB3RLEO?style=social)](https://github.com/CYB3RLEO)
