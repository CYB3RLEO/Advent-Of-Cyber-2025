# 🎄 TryHackMe Advent of Cyber 2025 - Day 19: ICS/Modbus - Claus for Concern

![TryHackMe](https://img.shields.io/badge/TryHackMe-AoC%202025-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Duration](https://img.shields.io/badge/Duration-60%20Minutes-blue?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-ICS%2FModbus-purple?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-OT%20Security-darkgreen?style=for-the-badge)

---

## 🎯 Overview

**Room**: Advent of Cyber 2025 - Day 19  
**Title**: ICS/Modbus - Claus for Concern  
**Topic**: Industrial Control Systems Security & Modbus Protocol  
**Difficulty**: Medium  
**Duration**: 60 minutes  
**Target**: Compromised SCADA/PLC system controlling drone deliveries

This challenge introduces **Industrial Control Systems (ICS)** security, teaching how to investigate and remediate attacks on SCADA systems using the Modbus protocol. Learn to identify configuration tampering, understand trap mechanisms, and safely restore critical infrastructure.

---

## 📖 The Story

### 🐰 The Easter Egg Attack

Heavy snow falls over Wareville as chaos erupts at TBFC headquarters. What should be the busiest shipping day has turned into a disaster—delivery drones are successfully completing their routes, but delivering **chocolate eggs** instead of Christmas presents!

The command center shows everything is normal:
- ✅ 1,000 presents in stock
- ✅ 98% success rate
- ✅ All systems operational

But citizens are receiving Easter baskets instead of the toys they ordered. Same weight, exact dimensions, but completely wrong items.

Then a message flashes on screen:

```
🐰 EGGSPLOIT v6.66 - Property of HopSec Island 🐰
"Why should Christmas have all the fun?" - King Malhare
```

Someone has compromised the drone fleet's control systems. This is a calculated assault on Christmas itself, and King Malhare has left traps for anyone who tries to fix it.

### 🔑 A Critical Discovery

You find a crumpled note near the PLC terminal:

```
TBFC DRONE CONTROL - REGISTER MAP
(For maintenance use only)

HOLDING REGISTERS:
HR0: Package Type Selection (0=Gifts, 1=Eggs, 2=Baskets)
HR1: Delivery Zone (1-9 normal, 10=ocean dump!)
HR4: System Signature/Version

COILS (Boolean Flags):
C10: Inventory Verification
C11: Protection/Override
C12: Emergency Dump Protocol
C13: Audit Logging
C14: Christmas Restored Flag
C15: Self-Destruct Status

⚠️ CRITICAL: Never change HR0 while C11=True!
Will trigger countdown!

- Maintenance Tech, Dec 19
```

This note will save Christmas...

---

## 🎓 Learning Objectives

By completing this challenge, you will:

- ✅ Understand how **SCADA systems** monitor industrial processes
- ✅ Learn what **PLCs** do in automation environments
- ✅ Master **Modbus protocol** communication and security weaknesses
- ✅ Identify **compromised configurations** in ICS systems
- ✅ Apply **safe remediation techniques** for critical infrastructure
- ✅ Understand **trap mechanisms** and defensive logic in ICS
- ✅ Recognize real-world ICS attack patterns (FrostyGoop malware)

---

## 🏭 SCADA Systems Explained

### What is SCADA?

**SCADA (Supervisory Control and Data Acquisition)** = The "nervous system" of industrial operations

**Function**: Bridge between human operators and physical machinery

### Four Key Components

| Component | Purpose | Example in TBFC System |
|-----------|---------|------------------------|
| **Sensors & Actuators** | Eyes and hands | Weight sensors, robotic arms loading drones |
| **PLCs** | Brains executing logic | Decision-making: which package, which drone |
| **Monitoring Systems** | Visual interfaces | CCTV cameras on port 80 showing warehouse floor |
| **Historians** | Data storage | Logs of every package, drone, system change |

### Why SCADA Systems Are Targeted

1. 🕰️ **Legacy software** with known vulnerabilities (decades old)
2. 🔓 **Default credentials** never changed
3. 🔧 **Designed for reliability, not security** (pre-cybersecurity era)
4. 🌍 **Control physical processes** (real-world consequences)
5. 🌐 **Connected to networks** (air-gap myth is fiction)
6. ⚠️ **Protocols lack authentication** (Modbus has no security)

**Real-world example**: **FrostyGoop (2024)** - First ICS/OT malware directly interfacing with Modbus TCP over port 502

---

## 🤖 PLC & Modbus Protocol

### What is a PLC?

**PLC (Programmable Logic Controller)** = Industrial computer controlling machinery

**Design characteristics**:
- 💪 Survive harsh environments (extreme temps, vibration, dust)
- ♾️ Run continuously for years without rebooting
- ⚡ Execute real-time control logic (millisecond responses)
- 🔌 Interface directly with physical hardware

### What is Modbus?

**Modbus** = Communication protocol for industrial devices (created 1979)

**Simple request-response**:
- Client: "PLC, what's register 0?"
- Server: "Register 0 = 1"

#### ⚠️ Security Problems

| Security Feature | Modbus Support |
|------------------|----------------|
| Authentication | ❌ None |
| Encryption | ❌ None |
| Authorization | ❌ None |
| Integrity Checking | ❌ None |

**Analogy**: Leaving your house unlocked with a sign: "Come in, everything's accessible!"

### Modbus Data Types

| Type | Purpose | Values | Read/Write |
|------|---------|--------|------------|
| **Coils** | Digital outputs | 0 or 1 | ✏️ Writable |
| **Discrete Inputs** | Digital inputs | 0 or 1 | 👁️ Read-only |
| **Holding Registers** | Analog outputs | 0-65535 | ✏️ Writable |
| **Input Registers** | Analog inputs | 0-65535 | 👁️ Read-only |

### TBFC System Configuration

#### Holding Registers (HR)

| Address | Purpose | Values | Default |
|---------|---------|--------|---------|
| **HR0** | Package type | 0=Gifts, 1=Eggs, 2=Baskets | 0 |
| **HR1** | Delivery zone | 1-9=normal, 10=ocean | 1-9 |
| **HR4** | System signature | Version ID | 100 |

#### Coils (C)

| Address | Purpose | Safe Value |
|---------|---------|------------|
| **C10** | Inventory verification | True |
| **C11** | Protection mechanism | False (to make changes) |
| **C12** | Emergency dump | False |
| **C13** | Audit logging | True |
| **C14** | Christmas restored | Auto-set |
| **C15** | Self-destruct | False |

### Modbus TCP vs Serial

**Original**: Serial connections (RS-232/RS-485) = Physical isolation
**Modern**: Modbus TCP = Listens on **port 502** = Network-accessible

**King Malhare's exploit**: Connected to port 502, issued commands as authorized user (no authentication!)

---

## 🚀 Complete Walkthrough

### Phase 1: Initial Reconnaissance

#### Step 1: Network Scanning

```bash
nmap -sV -p 22,80,502 MACHINE_IP
```

**Results**:
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1
80/tcp   open  http    Werkzeug httpd 3.1.3
502/tcp  open  modbus  Modbus TCP
```

**Key findings**:
- 🎥 **Port 80**: HTTP (CCTV camera feed)
- 🔧 **Port 502**: Modbus TCP (PLC communication)

#### Step 2: Visual Confirmation

Navigate to: `http://MACHINE_IP`

**Observations**:
- ✅ Robotic arms working smoothly
- ✅ Conveyor belts running
- ❌ **Pastel chocolate eggs** being sorted
- ❌ **Easter packaging** on assembly line
- ❌ Drones loading eggs instead of gifts
- ❌ Status: **"Compromised"**

**Analysis**: Not a system failure—logic manipulation attack!

Keep CCTV feed open for real-time feedback during remediation.

---

### Phase 2: Modbus Investigation

#### Step 1: Install PyModbus (if needed)

```bash
pip3 install pymodbus==3.6.8
```

#### Step 2: Connect to PLC

```python
python3

from pymodbus.client import ModbusTcpClient

# Connect to PLC on port 502
client = ModbusTcpClient('MACHINE_IP', port=502)

if client.connect():
    print("Connected to PLC successfully")
```

**Note**: No authentication required! 🚨

#### Step 3: Read Holding Registers

**Check HR0 (Package Type)**:

```python
result = client.read_holding_registers(address=0, count=1, slave=1)

if not result.isError():
    package_type = result.registers[0]
    print(f"HR0 (Package Type): {package_type}")
    # 0=Christmas, 1=Eggs, 2=Baskets
```

**Result**: `HR0 = 1` (Chocolate Eggs) 🐰

**Check HR1 (Delivery Zone)**:

```python
result = client.read_holding_registers(address=1, count=1, slave=1)
zone = result.registers[0]
print(f"HR1 (Delivery Zone): {zone}")
```

**Result**: `HR1 = 5` (Normal zone) ✅

**Check HR4 (System Signature)**:

```python
result = client.read_holding_registers(address=4, count=1, slave=1)
signature = result.registers[0]
print(f"HR4 (System Signature): {signature}")
```

**Result**: `HR4 = 666` ⚠️ **EGGSPLOIT SIGNATURE!**

#### Step 4: Read Coils

**Check C10 (Inventory Verification)**:

```python
result = client.read_coils(address=10, count=1, slave=1)
verification = result.bits[0]
print(f"C10 (Inventory Verification): {verification}")
```

**Result**: `C10 = False` (Disabled - not checking stock!) ❌

**Check C11 (Protection/Override)** - **CRITICAL**:

```python
result = client.read_coils(address=11, count=1, slave=1)
protection = result.bits[0]
print(f"C11 (Protection/Override): {protection}")
```

**Result**: `C11 = True` ⚠️ **TRAP IS ARMED!**

**Check C15 (Self-Destruct)**:

```python
result = client.read_coils(address=15, count=1, slave=1)
armed = result.bits[0]
print(f"C15 (Self-Destruct Armed): {armed}")
```

**Result**: `C15 = False` (Not armed yet, but will trigger if we change HR0 while C11=True!)

---

### Phase 3: Understanding The Trap

**The Trap Mechanism**:

1. 🎯 HR0 set to 1 (forcing eggs)
2. 🔒 C11 enabled (monitoring for changes)
3. ⚠️ **Changing HR0 while C11=True → Arms C15**
4. ⏱️ C15 armed → 30-second countdown
5. 💥 After 30 seconds → C12 activates (Emergency Dump)
6. 🌊 Everything dumps to Zone 10 (ocean)

**Maintenance tech's warning**: "Never change HR0 while C11=True!"

---

### Phase 4: Complete Reconnaissance Script

Create `reconnaissance.py`:

```python
#!/usr/bin/env python3
from pymodbus.client import ModbusTcpClient

PLC_IP = "MACHINE_IP"
PORT = 502
UNIT_ID = 1

client = ModbusTcpClient(PLC_IP, port=PORT)

if not client.connect():
    print("Failed to connect to PLC")
    exit(1)

print("=" * 60)
print("TBFC Drone System - Reconnaissance Report")
print("=" * 60)

# Read holding registers
registers = client.read_holding_registers(address=0, count=5, slave=UNIT_ID)
if not registers.isError():
    hr0, hr1, hr2, hr3, hr4 = registers.registers
    print(f"\nHOLDING REGISTERS:")
    print(f"HR0 (Package Type): {hr0} (0=Gifts, 1=Eggs, 2=Baskets)")
    print(f"HR1 (Delivery Zone): {hr1} (1-9=Normal, 10=Ocean)")
    print(f"HR4 (System Signature): {hr4}")
    if hr4 == 666:
        print(f"  ⚠️  EGGSPLOIT DETECTED")

# Read coils
coils = client.read_coils(address=10, count=6, slave=UNIT_ID)
if not coils.isError():
    c10, c11, c12, c13, c14, c15 = coils.bits[:6]
    print(f"\nCOILS:")
    print(f"C10 (Inventory Verification): {c10} (Should be True)")
    print(f"C11 (Protection/Override): {c11}")
    if c11:
        print(f"  ⚠️  TRAP IS ARMED")
    print(f"C12 (Emergency Dump): {c12}")
    print(f"C13 (Audit Logging): {c13} (Should be True)")
    print(f"C14 (Christmas Restored): {c14}")
    print(f"C15 (Self-Destruct): {c15}")

print("\n" + "=" * 60)
print("THREAT ASSESSMENT:")
print("=" * 60)
if hr4 == 666:
    print("✗ Eggsploit framework detected")
if c11:
    print("✗ Protection mechanism active - trap is set")
if hr0 == 1:
    print("✗ Package type forced to eggs")
if not c10:
    print("✗ Inventory verification disabled")

print("\n⚠️  REMEDIATION REQUIRED")

client.close()
```

Run it:

```bash
python3 reconnaissance.py
```

---

### Phase 5: Safe Remediation

#### ⚠️ Critical Order of Operations

Must follow this exact sequence:

1. ✅ **Disable C11 (Protection)** ← FIRST!
2. ✅ Change HR0 to 0 (Christmas gifts)
3. ✅ Enable C10 (Inventory verification)
4. ✅ Enable C13 (Audit logging)
5. ✅ Verify C15 never armed

**Why order matters**: Changing HR0 before disabling C11 = 💥 TRAP TRIGGERS

#### Restoration Script

Create `restore_christmas.py`:

```python
#!/usr/bin/env python3
from pymodbus.client import ModbusTcpClient
import time

PLC_IP = "MACHINE_IP"
PORT = 502
UNIT_ID = 1

def read_coil(client, address):
    result = client.read_coils(address=address, count=1, slave=UNIT_ID)
    if not result.isError():
        return result.bits[0]
    return None

def read_register(client, address):
    result = client.read_holding_registers(address=address, count=1, slave=UNIT_ID)
    if not result.isError():
        return result.registers[0]
    return None

# Connect to PLC
client = ModbusTcpClient(PLC_IP, port=PORT)

if not client.connect():
    print("Failed to connect to PLC")
    exit(1)

print("=" * 60)
print("TBFC Drone System - Christmas Restoration")
print("=" * 60)

# Step 1: Verify current state
print("\nStep 1: Verifying current system state...")
time.sleep(1)
package_type = read_register(client, 0)
protection = read_coil(client, 11)
armed = read_coil(client, 15)
print(f"  Package Type: {package_type} (1 = Eggs)")
print(f"  Protection Active: {protection}")
print(f"  Self-Destruct Armed: {armed}")

# Step 2: Disable protection (CRITICAL FIRST STEP!)
print("\nStep 2: Disabling protection mechanism...")
time.sleep(1)
result = client.write_coil(11, False, slave=UNIT_ID)
if not result.isError():
    print("  ✅ Protection DISABLED")
    print("  Safe to proceed with changes")
else:
    print("  ❌ FAILED to disable protection")
    exit(1)

# Step 3: Change package type to Christmas
print("\nStep 3: Setting package type to Christmas presents...")
time.sleep(1)
result = client.write_register(0, 0, slave=UNIT_ID)
if not result.isError():
    print("  ✅ Package type changed to: Christmas Presents")

# Step 4: Enable inventory verification
print("\nStep 4: Enabling inventory verification...")
time.sleep(1)
result = client.write_coil(10, True, slave=UNIT_ID)
if not result.isError():
    print("  ✅ Inventory verification ENABLED")

# Step 5: Enable audit logging
print("\nStep 5: Enabling audit logging...")
time.sleep(1)
result = client.write_coil(13, True, slave=UNIT_ID)
if not result.isError():
    print("  ✅ Audit logging ENABLED")

# Step 6: Verify restoration
print("\nStep 6: Verifying system restoration...")
time.sleep(2)
christmas_restored = read_coil(client, 14)
new_package_type = read_register(client, 0)
emergency_dump = read_coil(client, 12)
self_destruct = read_coil(client, 15)

print(f"  Package Type: {new_package_type} (0 = Christmas)")
print(f"  Christmas Restored: {christmas_restored}")
print(f"  Emergency Dump: {emergency_dump}")
print(f"  Self-Destruct Armed: {self_destruct}")

if christmas_restored and new_package_type == 0 and not self_destruct:
    print("\n" + "=" * 60)
    print("🎄 SUCCESS - CHRISTMAS IS SAVED 🎄")
    print("=" * 60)
    
    # Read the flag from registers
    flag_result = client.read_holding_registers(address=20, count=12, slave=UNIT_ID)
    if not flag_result.isError():
        flag_bytes = []
        for reg in flag_result.registers:
            flag_bytes.append(reg >> 8)
            flag_bytes.append(reg & 0xFF)
        flag = ''.join(chr(b) for b in flag_bytes if b != 0)
        print(f"\nFlag: {flag}")
    
    print("\nChristmas deliveries have been restored!")
    print("Check the CCTV feed to see the results")
else:
    print("\n❌ Restoration incomplete - check system state")

client.close()
print("\nDisconnected from PLC")
```

#### Execute Restoration

```bash
python3 restore_christmas.py
```

**Success Output**:

```
============================================================
🎄 SUCCESS - CHRISTMAS IS SAVED 🎄
============================================================

Flag: THM{eGgMas0V3r}

Christmas deliveries have been restored!
Check the CCTV feed to see the results
```

Check CCTV at `http://MACHINE_IP` - King Malhare defeated! 🎉

---

## 🎯 Flags & Answers

| Question | Answer |
|----------|--------|
| What port is commonly used by Modbus TCP? | `502` |
| What's the flag? | `THM{eGgMas0V3r}` |

---

## 📚 Command Reference

### Network Scanning

```bash
# Quick scan of specific ports
nmap -sV -p 22,80,502 MACHINE_IP

# Comprehensive all-port scan (slower)
nmap -sV -T4 -p- -vv MACHINE_IP
```

### Python PyModbus Operations

| Operation | Code |
|-----------|------|
| **Install library** | `pip3 install pymodbus==3.6.8` |
| **Create client** | `client = ModbusTcpClient(IP, port=502)` |
| **Connect** | `client.connect()` |
| **Read holding register** | `client.read_holding_registers(address, count, slave=1)` |
| **Write holding register** | `client.write_register(address, value, slave=1)` |
| **Read coil** | `client.read_coils(address, count, slave=1)` |
| **Write coil** | `client.write_coil(address, value, slave=1)` |
| **Close connection** | `client.close()` |

### Register/Coil Address Reference

| Address | Type | Purpose | Safe Value |
|---------|------|---------|------------|
| **0** | Holding Register | Package type | 0 (Christmas) |
| **1** | Holding Register | Delivery zone | 1-9 (normal zones) |
| **4** | Holding Register | System signature | 100 (default) |
| **10** | Coil | Inventory verification | True |
| **11** | Coil | Protection mechanism | False (to make changes) |
| **12** | Coil | Emergency dump | False |
| **13** | Coil | Audit logging | True |
| **14** | Coil | Christmas restored | Auto-set when fixed |
| **15** | Coil | Self-destruct | False |

---

## ⏱️ Attack Timeline

### Pre-Attack (Normal Operations)

```
HR0 = 0 (Christmas gifts)
HR1 = 5 (Normal zone)
HR4 = 100 (Default signature)
C10 = True (Verification enabled)
C11 = False (Protection disabled)
C13 = True (Logging enabled)
Status: ✅ Operational
```

### King Malhare's Attack Sequence

```
Step 1: Connect to port 502 (no authentication required)
Step 2: Change HR0 = 1 (Force egg deliveries)
Step 3: Disable C10 = False (Turn off inventory verification)
Step 4: Disable C13 = False (Turn off audit logging)
Step 5: Set HR4 = 666 (Leave Eggsploit signature)
Step 6: Enable C11 = True (Arm trap mechanism)
Result: 🐰 System delivering eggs, trap armed
```

### Trap Mechanism (If Triggered)

```
Incorrect Remediation:
  → Change HR0 while C11=True
  → C15 arms immediately
  → 30-second countdown begins
  → C12 activates (Emergency dump)
  → HR1 changes to 10 (Ocean zone)
  → All inventory dumped
Result: 🌊 System destroyed, must restart challenge
```

### Correct Remediation Sequence

```
Step 1: Disable C11 = False (Disarm trap)
Step 2: Change HR0 = 0 (Restore Christmas gifts)
Step 3: Enable C10 = True (Enable verification)
Step 4: Enable C13 = True (Enable logging)
Step 5: Verify C14 = True (Christmas restored)
Step 6: Verify C15 = False (Self-destruct disarmed)
Result: 🎄 Christmas saved!
```

---

## 🔐 Key Security Concepts

### ICS vs IT Security Differences

| Aspect | Traditional IT | ICS/OT |
|--------|----------------|--------|
| **Priority** | Confidentiality | Availability |
| **Updates** | Regular patches | Rare (stability critical) |
| **Downtime** | Tolerable | Unacceptable |
| **Consequences** | Data breach | Physical damage |
| **Protocols** | Modern, secure | Legacy, insecure |
| **Authentication** | Standard | Often absent |

### Why Modbus is Vulnerable

```
❌ No authentication → Anyone can connect
❌ No encryption → All traffic visible
❌ No authorization → Any command allowed
❌ No integrity → Commands can be altered
❌ No logging → Actions unrecorded
```

**Modern solutions** (add-ons, not built-in):
- VPNs for remote access
- Modbus security gateways
- SSL/TLS wrappers
- Network segmentation
- Firewalls and IDS

### Real-World ICS Attacks

| Attack | Year | Target | Method |
|--------|------|--------|--------|
| **Stuxnet** | 2010 | Iranian nuclear | Siemens PLCs |
| **BlackEnergy** | 2015 | Ukraine power grid | SCADA compromise |
| **Triton/Trisis** | 2017 | Petrochemical | Safety systems |
| **FrostyGoop** | 2024 | ICS/OT systems | Modbus TCP (port 502) |

### Defense Strategies

#### 1. Network Segmentation
```
Business Network ←→ [Firewall/DMZ] ←→ SCADA Network
                                      ↓
                                   Port 502 (Restricted)
```

#### 2. Access Controls
- Implement authentication layers
- Use VPNs for remote access
- Whitelist authorized IP addresses
- Deploy Modbus security gateways

#### 3. Monitoring & Logging
- Enable audit logging on all systems
- Monitor Modbus traffic for anomalies
- Alert on unexpected register/coil changes
- Maintain historian databases

#### 4. Change Management
- Document all configurations
- Require approval for modifications
- Test changes in isolated environments
- Maintain backup configurations

#### 5. Incident Response
- Develop ICS-specific response plans
- Train on safe remediation procedures
- **Understand trap mechanisms before fixing**
- Maintain visual monitoring during remediation

---

## 💡 Key Takeaways

✅ **SCADA systems** are command centers bridging operators and physical machinery

✅ **PLCs** are ruggedized computers for continuous operation in harsh environments

✅ **Modbus** is widely deployed but has **zero built-in security**

✅ **Port 502** is default for Modbus TCP and often unauthenticated

✅ **Holding Registers** store configuration; **Coils** control boolean flags

✅ **Trap mechanisms** require careful analysis before remediation

✅ **Order of operations matters** critically when restoring systems

✅ **Visual monitoring** (CCTV) provides real-time feedback

✅ **ICS security** has physical world consequences

✅ **Understanding before action** isn't optional—it's essential

---

## 🔗 Related Rooms

Continue your ICS security journey:

- **Industrial Intrusion**: OT/ICS security deep dive
- **ICS Security Basics**: Foundational ICS knowledge
- **SCADA and ICS Security**: Advanced SCADA exploitation
- **Malware Analysis**: Analyzing ICS-targeted malware
- **Network Security**: Segmentation and monitoring

---

## 🤝 Contributing

Found an issue or want to improve this walkthrough?

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

---

## 📄 License

This walkthrough is created for educational purposes as part of TryHackMe's Advent of Cyber 2025.

---

## 🙏 Acknowledgments

- **TryHackMe** for creating Advent of Cyber 2025
- The ICS security community for sharing knowledge
- Maintenance technicians everywhere who document their systems
- Security researchers studying industrial control systems


---

**🎄 Christmas Saved! Happy Hacking! 🔐**

*In ICS environments, understanding before action isn't just best practice—it's essential.*

---

## ⚠️ Important Warning

**What If You Trigger The Trap?**

If you change HR0 before disabling C11:

```
⚠️  C15 (Self-Destruct) arms immediately
⏱️  30-second countdown begins
💥 C12 (Emergency Dump) activates
🌊 HR1 → 10 (Ocean zone)
📦 All inventory dumps
🔄 Challenge must be restarted
```

**Lesson**: In real-world ICS, triggering safety mechanisms could cause:
- Power outages
- Contaminated water supplies
- Industrial accidents
- Physical harm

**Always analyze before acting!**
