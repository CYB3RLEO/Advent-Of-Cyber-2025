# 🎄 Advent of Cyber 2025 - Day 1: Linux CLI - Shells & Bells

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-30%20min-blue?style=for-the-badge)](https://tryhackme.com)

## 📖 Overview

Day 1 of TryHackMe's Advent of Cyber 2025 introduces the Linux command-line interface through an engaging investigation. McSkidy has been kidnapped, and we must use CLI commands to uncover a security breach at The Best Festival Company (TBFC).

### 🎯 Learning Objectives

- Master basic Linux CLI commands
- Navigate the Linux file system
- Search and analyze log files
- Discover and analyze hidden files
- Investigate bash history for security traces

## 🔍 Investigation Summary

**Threat Actor:** Sir Carrotbane & HopSec Island  
**Target:** TBFC SOC-mas Platform  
**Attack Vector:** Brute force + Malicious script deployment  
**Impact:** Christmas wishlist theft and replacement

## 📝 Walkthrough

### Initial Reconnaissance

```bash
# Verify CLI functionality
echo "Hello World!"

# List directory contents
ls

# Read McSkidy's README
cat README.txt
```

### Finding the Hidden Guide

```bash
# Navigate to Guides directory
cd Guides

# List all files including hidden ones
ls -la

# Read the hidden security guide
cat .guide.txt
```

**🚩 Flag 1:** `THM{learning-linux-cli}`

### Log Analysis

```bash
# Navigate to log directory
cd /var/log

# Search for failed login attempts
grep "Failed password" auth.log
```

**Discovery:** Multiple failed logins from `eggbox-196.hopsec.thm`

### Malware Discovery

```bash
# Search for suspicious files
find /home/socmas -name *egg*

# Navigate to the suspicious directory
cd /home/socmas/2025

# Analyze the malicious script
cat eggstrike.sh
```

**🚩 Flag 2:** `THM{sir-carrotbane-attacks}`

### Privilege Escalation & History Analysis

```bash
# Switch to root user
sudo su

# Verify current user
whoami

# Navigate to root home
cd /root

# Check bash history
cat .bash_history
# or
history
```

**🚩 Flag 3:** `THM{until-we-meet-again}`

## 🛠️ Essential Commands Reference

| Command | Description | Example |
|---------|-------------|---------|
| `ls` | List directory contents | `ls -la` |
| `cd` | Change directory | `cd /var/log` |
| `cat` | Display file contents | `cat file.txt` |
| `grep` | Search for patterns | `grep "error" log.txt` |
| `find` | Search for files | `find / -name "*.txt"` |
| `pwd` | Print working directory | `pwd` |
| `whoami` | Show current user | `whoami` |
| `sudo su` | Switch to root user | `sudo su` |
| `history` | Show command history | `history` |
| `echo` | Display text/variables | `echo "Hello"` |

## 🔐 Security Concepts

### Log Analysis
Examining authentication logs (`/var/log/auth.log`) to identify failed login attempts and potential brute-force attacks.

### Hidden Files
Files starting with `.` are hidden in Linux. Attackers often use this to conceal malware.

### Bash History
Command history stored in `~/.bash_history` can reveal attacker actions and TTPs.

### Privilege Escalation
Root access provides complete system control, making it a prime target for attackers.

## 📊 Attack Timeline

1. **Initial Access:** Failed brute-force attempts on `socmas` account
2. **Script Deployment:** `eggstrike.sh` placed in `/home/socmas/2025/`
3. **Data Exfiltration:** Wishlist data sent to `files.hopsec.thm`
4. **Data Manipulation:** Christmas wishlist replaced with EASTMAS version
5. **Cleanup:** Traces left in root bash history

## 🎓 Key Takeaways

- CLI proficiency is essential for cybersecurity professionals
- Log files are crucial for incident investigation
- Hidden files can conceal malicious activity
- Bash history provides valuable forensic evidence
- Understanding special characters (`|`, `>`, `&&`) enhances CLI efficiency

## 🏆 Challenge Answers

1. **Which CLI command would you use to list a directory?**  
   Answer: `ls`

2. **What flag did you see inside of McSkidy's guide?**  
   Answer: `THM{learning-linux-cli}`

3. **Which command helped you filter the logs for failed logins?**  
   Answer: `grep`

4. **What flag did you see inside the Eggstrike script?**  
   Answer: `THM{sir-carrotbane-attacks}`

5. **Which command would you run to switch to the root user?**  
   Answer: `sudo su`

6. **What flag did Sir Carrotbane leave in the root bash history?**  
   Answer: `THM{until-we-meet-again}`

## 🎁 Bonus Content

For intermediate users, explore McSkidy's hidden note in `/home/mcskidy/Documents/` to unlock Side Quest 1!

## 🔗 Related Rooms

- [Linux Fundamentals](https://tryhackme.com/room/linuxfundamentalspart1)
- [Linux Logs Investigations](https://tryhackme.com)
- [Linux Privilege Escalation](https://tryhackme.com/room/linuxprivesc)

## 📚 Additional Resources

- [Linux Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners)
- [Grep Command Tutorial](https://www.gnu.org/software/grep/manual/grep.html)
- [Linux File System Hierarchy](https://www.pathname.com/fhs/)

## 🤝 Contributing

Found an error or have suggestions? Feel free to open an issue or submit a pull request!

## 📜 License

This walkthrough is for educational purposes only. All credit goes to TryHackMe and the Advent of Cyber 2025 team.

---

⭐ **Star this repo if you found it helpful!**

🎄 **Happy Hacking & Merry Christmas!**

[![Follow on GitHub](https://img.shields.io/github/followers/CYB3RLEO?style=social)](https://github.com/CYB3RLEO)
[![Twitter Follow](https://img.shields.io/x/follow/_cyb3rleo?style=social)](https://x.com/_cyb3rleo)

---

**Next:** [Day 2 - Coming Soon!](../day-02/)
