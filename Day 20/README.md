# 🎄 TryHackMe Advent of Cyber 2025 - Day 20: Race Conditions - Toy to The World

![TryHackMe](https://img.shields.io/badge/TryHackMe-AoC%202025-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)
![Duration](https://img.shields.io/badge/Duration-30%20Minutes-blue?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Race%20Conditions-orange?style=for-the-badge)
![Tool](https://img.shields.io/badge/Tool-Burp%20Suite-purple?style=for-the-badge)



---

## 🎯 Overview

**Room**: Advent of Cyber 2025 - Day 20  
**Title**: Race Conditions - Toy to The World  
**Topic**: Exploiting Race Conditions in Web Applications  
**Difficulty**: Easy  
**Duration**: 30 minutes  
**Tools**: Burp Suite, Firefox, AttackBox

Learn how to exploit race condition vulnerabilities to oversell limited-edition items in an e-commerce application, understand the three types of race conditions, and implement proper mitigation strategies.

---

## 📖 The Story

### 🎁 The Midnight Launch That Went Wrong

The Best Festival Company (TBFC) launched its highly anticipated **limited-edition SleighToy** at midnight—only **10 pieces** available worldwide.

Within seconds, thousands rushed to buy one. Then something impossible happened: **More than ten customers** received order confirmation emails. 

Confusion spread across social media:
- "I got order #15 for the last toy!"
- "How did I buy item 12 of 10?"
- "The website shows negative stock!"

**McSkidy was called in to investigate.**

She discovered the culprit: multiple buyers purchasing at the **exact same moment**, exploiting a critical timing flaw. Sir Carrotbane's **Bandit Bunnies** had weaponized this vulnerability, flooding checkouts with simultaneous requests.

By morning:
- 😠 Angry shoppers demanding confirmed orders
- 📦 Impossible inventory numbers (negative stock)
- 🐰 Evidence of deliberate exploitation
- ⏱️ A timing-based attack that revealed system weaknesses

**The vulnerability**: A **race condition**—when multiple operations execute simultaneously without proper synchronization.

---

## 🎓 Learning Objectives

By completing this challenge, you will:

- ✅ Understand what **race conditions** are and their impact
- ✅ Learn to **identify and exploit** race conditions in web requests
- ✅ Master **Burp Suite** for sending parallel requests
- ✅ See how **concurrent requests** manipulate stock values
- ✅ Apply **mitigation techniques** to prevent race conditions
- ✅ Understand real-world e-commerce security implications

---

## 🔄 Understanding Race Conditions

### What is a Race Condition?

A **race condition** occurs when:
- Two or more operations execute **simultaneously**
- The system's outcome depends on the **order** they finish
- Without proper synchronization, unexpected results occur

**Real-world analogy**: Two people trying to buy the last concert ticket simultaneously. Both check availability, both see "1 available", both click "Buy"—if the system doesn't handle this properly, **both succeed** even though only one ticket exists.

### Consequences Without Proper Synchronization

| Issue | Description | Example |
|-------|-------------|---------|
| **Duplicate Transactions** | Same operation executed multiple times | Double payment processing |
| **Oversold Items** | More sold than available | Negative inventory |
| **Unauthorized Changes** | Data modified unexpectedly | Account balance manipulation |
| **Inconsistent States** | System in invalid state | Order confirmed but payment failed |

---

## 🏷️ Types of Race Conditions

### 1️⃣ Time-of-Check to Time-of-Use (TOCTOU)

**Definition**: Data checked at one point is used later, but changes occur between check and use.

**Flow**:
```
User A: Check stock → 10 available
User B: Check stock → 10 available
User A: Purchase → Stock = 9
User B: Purchase → Stock = 8
```

**If simultaneous**:
```
Users A & B: Check stock → BOTH see 10
Users A & B: Purchase → BOTH succeed
Result: Stock = 9 (should be 8!)
```

**Real-world example**: Checking parking space availability, but it's taken by the time you arrive.

---

### 2️⃣ Shared Resource Race Condition

**Definition**: Multiple users modify the same data simultaneously without locks.

**Vulnerable code pattern**:
```python
# Thread 1 and Thread 2 execute simultaneously
current_stock = get_stock()  # Both read: 10
new_stock = current_stock - 1  # Both calculate: 9
set_stock(new_stock)  # Both write: 9

# Expected: 8 (two sales)
# Actual: 9 (one sale counted)
```

**Real-world example**: Two cashiers updating the same spreadsheet—one overwrites the other's work.

---

### 3️⃣ Atomicity Violation

**Definition**: Multi-step operation should be atomic (all-or-nothing), but steps execute separately.

**Vulnerable flow**:
```
Step 1: Deduct from inventory → SUCCESS
[Another request interrupts here]
Step 2: Create order record → FAIL

Result: Inventory decreased, but no order exists
```

**Real-world example**: Vending machine takes your money, then loses power before dispensing the product.

---

## 🚀 Complete Walkthrough

### Phase 1: Environment Setup

#### Step 1: Configure Firefox Proxy

```
1. Open Firefox on AttackBox
2. Click FoxyProxy icon (top-right)
3. Select "Burp" profile
```

**Purpose**: Routes all browser traffic through Burp Suite.

#### Step 2: Launch Burp Suite

```
1. Click Burp Suite icon on Desktop
2. Select "Temporary project in memory" → Next
3. Click "Start Burp"
4. Wait for welcome screen
```

#### Step 3: Disable Intercept Mode ⚠️

**Critical step**:
```
1. Open Proxy tab
2. Click Intercept sub-tab
3. Ensure button says "Intercept is off"
```

**Why**: Prevents Burp from blocking requests while still logging them.

---

### Phase 2: Legitimate Purchase

#### Step 1: Access Web Application

```
URL: http://MACHINE_IP
```

#### Step 2: Login

```
Username: attacker
Password: attacker@123
```

#### Step 3: Complete Purchase Flow

1. View **SleighToy Limited Edition** (10 units available)
2. Click **"Add to Cart"**
3. Click **"Checkout"**
4. Click **"Confirm & Pay"**
5. Receive success confirmation

**Purpose**: This creates the POST request to `/process_checkout` that we'll exploit.

---

### Phase 3: Exploit Race Condition

#### Step 1: Capture Checkout Request

```
1. Navigate to Burp Suite
2. Click: Proxy → HTTP history
3. Find: POST request to /process_checkout
4. Right-click → "Send to Repeater"
```

**What you captured**: The exact HTTP request with headers, cookies, and body parameters.

#### Step 2: Create Request Group

```
1. Switch to Repeater tab
2. Right-click request tab
3. Select "Add tab to group"
4. Click "Create tab group"
5. Name it: cart
6. Click "Create"
```

#### Step 3: Duplicate Requests

```
1. Right-click request tab
2. Select "Duplicate tab"
3. Enter number: 15
4. Confirm
```

**Result**: 15 identical requests ready to execute.

**Why 15?** Enough parallel requests to overwhelm the system's synchronization (or lack thereof).

#### Step 4: Configure Parallel Sending

```
1. Click "Send" dropdown in toolbar
2. Select "Send group in parallel (last-byte sync)"
```

**What this does**: 
- Sends all 15 requests simultaneously
- Waits for final byte from each response
- Maximizes timing overlap

#### Step 5: Execute Attack

```
1. Click "Send group (parallel)"
2. All 15 requests launch at once
3. Server attempts to process simultaneously
```

**Behind the scenes**:
```
All 15 requests:
  → Check stock: 10 available ✓
  → Create order ✓
  → Update stock (race happens here!)

Result: 15 orders, stock becomes negative
```

#### Step 6: Verify Exploitation

```
1. Return to web app: http://MACHINE_IP
2. Observe:
   - Multiple confirmed orders
   - Negative stock for SleighToy
   - Flag displayed
```

---

### Phase 4: Exploit Bunny Plush (Blue)

**Repeat the process**:

1. ✅ Add Bunny Plush (Blue) to cart
2. ✅ Complete checkout normally
3. ✅ Capture `/process_checkout` request in Burp
4. ✅ Send to Repeater
5. ✅ Create new tab group: `bunny`
6. ✅ Duplicate tab: 15 copies
7. ✅ Send group in parallel (last-byte sync)
8. ✅ Execute attack
9. ✅ Verify negative stock and retrieve flag

---

## 🎯 Flags & Answers

| Question | Answer |
|----------|--------|
| What is the flag value once the stocks are negative for SleighToy Limited Edition? | `THM{WINNER_OF_R@CE007}` |
| Repeat the same steps for Bunny Plush (Blue). What is the flag value once the stocks are negative? | `THM{WINNER_OF_Bunny_R@ce}` |

---

## ⚙️ Attack Mechanics

### Vulnerable Code Pattern

```python
def process_checkout(item_id, quantity):
    # VULNERABILITY: No locking mechanism
    
    # Step 1: Check stock
    current_stock = get_stock(item_id)
    
    # Step 2: Validate
    if current_stock >= quantity:
        # Step 3: Create order (takes time)
        create_order(item_id, quantity)
        
        # Step 4: Update stock (race window here!)
        new_stock = current_stock - quantity
        update_stock(item_id, new_stock)
        
        return "Order successful"
    
    return "Out of stock"
```

**The Problem**: Between checking stock and updating it, other requests can slip through.

### Attack Timeline

```
Time    Event
────────────────────────────────────────────────
0ms     Initial stock: 10
1ms     Request 1: Check stock → 10 ✓
2ms     Request 2: Check stock → 10 ✓
3ms     Request 3: Check stock → 10 ✓
...
15ms    Request 15: Check stock → 10 ✓

[All requests pass validation simultaneously]

100ms   Request 1: Update stock → 9
101ms   Request 2: Update stock → 8
102ms   Request 3: Update stock → 7
...
114ms   Request 15: Update stock → -5

────────────────────────────────────────────────
Result: 15 orders confirmed, stock = -5
Expected: Only 10 orders should succeed
```

### Why Parallel Requests Matter

**Sequential requests** (normal):
```
Request 1: Check (10) → Buy → Update (9)
Request 2: Check (9) → Buy → Update (8)
Request 3: Check (8) → Buy → Update (7)
...
Request 11: Check (0) → BLOCKED (out of stock)

Result: 10 orders, stock = 0 ✓ CORRECT
```

**Parallel requests** (race condition):
```
Requests 1-15: ALL check (10) simultaneously
All pass validation before any update
All succeed even though only 10 available

Result: 15 orders, stock = -5 ✗ EXPLOITED
```

---

## 🛡️ Mitigation Strategies

### 1. Atomic Database Transactions

**Purpose**: Ensure operations execute as single, consistent unit.

```python
def process_checkout(item_id, quantity):
    with database.transaction():
        # Lock the row during transaction
        current_stock = database.select_for_update(
            "SELECT stock FROM inventory WHERE id = ?", 
            item_id
        )
        
        if current_stock >= quantity:
            create_order(item_id, quantity)
            update_stock(item_id, current_stock - quantity)
            database.commit()
            return "Success"
        else:
            database.rollback()
            return "Out of stock"
```

**Key**: `SELECT ... FOR UPDATE` locks the row, blocking other transactions.

---

### 2. Final Stock Validation

**Purpose**: Last-second check before committing.

```python
def process_checkout(item_id, quantity):
    with database.transaction():
        current_stock = get_stock(item_id)
        
        if current_stock >= quantity:
            new_stock = current_stock - quantity
            
            # CRITICAL: Final validation
            if new_stock < 0:
                database.rollback()
                return "Out of stock"
            
            create_order(item_id, quantity)
            update_stock(item_id, new_stock)
            database.commit()
            return "Success"
```

---

### 3. Idempotency Keys

**Purpose**: Prevent duplicate request processing.

```python
def process_checkout(item_id, quantity, idempotency_key):
    # Check if request already processed
    if order_exists_with_key(idempotency_key):
        return "Already processed"
    
    with database.transaction():
        current_stock = get_stock_with_lock(item_id)
        
        if current_stock >= quantity:
            create_order(item_id, quantity, idempotency_key)
            update_stock(item_id, current_stock - quantity)
            database.commit()
            return "Success"
```

**Client-side**: Generate unique UUID for each checkout attempt.

---

### 4. Rate Limiting

**Purpose**: Block rapid repeated attempts.

```python
# User-level rate limit
if get_checkout_count(user_id, last_second) >= 1:
    return "Too many requests"

# System-wide concurrency limit
checkout_semaphore = Semaphore(5)  # Max 5 concurrent

def process_checkout(item_id, quantity):
    if not checkout_semaphore.acquire(timeout=5):
        return "System busy"
    
    try:
        # Process with atomic transaction
        pass
    finally:
        checkout_semaphore.release()
```

---

### 5. Optimistic Locking

**Purpose**: Detect concurrent modifications using version numbers.

```python
def process_checkout(item_id, quantity):
    # Read with version
    current_stock, version = get_stock_with_version(item_id)
    
    if current_stock >= quantity:
        new_stock = current_stock - quantity
        
        # Update only if version unchanged
        rows_updated = update_if_version_matches(
            item_id, new_stock, version, new_version=version+1
        )
        
        if rows_updated == 0:
            return "Conflict - please retry"
        
        create_order(item_id, quantity)
        return "Success"
```

---

## 🔑 Key Concepts

### Race Condition vs Other Vulnerabilities

| Vulnerability | Timing-Dependent? | Requires Concurrency? |
|---------------|-------------------|-----------------------|
| **Race Condition** | ✅ Yes | ✅ Yes |
| **SQL Injection** | ❌ No | ❌ No |
| **XSS** | ❌ No | ❌ No |
| **CSRF** | ❌ No | ❌ No |

**Key difference**: Race conditions **require concurrent execution** to exploit.

### Critical Timing Windows

```
┌────────────────────────────────────────┐
│ Check if stock available               │ ← SAFE
├────────────────────────────────────────┤
│                                        │
│  ⚠️  RACE CONDITION WINDOW  ⚠️         │ ← VULNERABLE
│                                        │
├────────────────────────────────────────┤
│ Update stock                           │ ← TOO LATE
└────────────────────────────────────────┘
```

**The window**: Between check and update, concurrent requests can slip through.

### Real-World Impact

#### E-Commerce Disasters

| Event | Impact | Cause |
|-------|--------|-------|
| **Black Friday overselling** | Thousands of cancelled orders | No concurrency controls |
| **Limited sneaker drops** | Bots buy 100+ pairs | Race condition exploitation |
| **Gaming console launches** | 10x oversold inventory | TOCTOU vulnerability |

#### Financial Consequences

- 💰 **Refunds**: Must refund oversold items
- 😠 **Reputation**: Customer trust destroyed
- ⚖️ **Legal**: Advertising standards violations
- 📉 **Revenue**: Items sold beyond intended limits

---

## 💡 Key Takeaways

✅ **Race conditions** occur when concurrent operations lack synchronization

✅ **Three types**: TOCTOU, Shared Resource, Atomicity Violation

✅ **TOCTOU** exploits the gap between checking and using data

✅ **Shared resource races** happen when multiple threads modify data simultaneously

✅ **Atomicity violations** occur when multi-step operations execute separately

✅ **Burp Suite** makes exploiting race conditions straightforward

✅ **Mitigation requires** atomic transactions, locking, idempotency, and rate limiting

✅ **Real-world impact** includes overselling, financial losses, reputation damage

✅ **E-commerce systems** are prime targets during high-traffic events

✅ **Testing under load** is essential before production deployment

---

## 📚 Command Reference

### Burp Suite Workflow

| Step | Action |
|------|--------|
| **1. Configure proxy** | Firefox → FoxyProxy → Select "Burp" |
| **2. Launch Burp** | Desktop icon → Temporary project → Start Burp |
| **3. Disable intercept** | Proxy → Intercept → "Intercept is off" |
| **4. Make request** | Browser → Login → Add to cart → Checkout |
| **5. Capture request** | Proxy → HTTP history → Find POST `/process_checkout` |
| **6. Send to Repeater** | Right-click → Send to Repeater |
| **7. Create group** | Repeater → Right-click tab → Add tab to group |
| **8. Duplicate** | Right-click → Duplicate tab → Enter 15 |
| **9. Configure parallel** | Send dropdown → "Send group in parallel (last-byte sync)" |
| **10. Execute** | Click "Send group (parallel)" |
| **11. Verify** | Return to web app → Check negative stock |

---

## 🔗 Related Rooms

Continue your race condition expertise:

- **Race Conditions**: Comprehensive race condition exploitation
- **Web Application Security**: Broader web vulnerability landscape
- **Burp Suite Basics**: Master web application testing tools
- **OWASP Top 10**: Common web security vulnerabilities
- **Secure Code Review**: Learn to spot vulnerabilities in code

---

## 🤝 Contributing

Improvements welcome!

1. Fork this repository
2. Create feature branch (`git checkout -b feature/improvement`)
3. Commit changes (`git commit -m 'Add improvement'`)
4. Push to branch (`git push origin feature/improvement`)
5. Open Pull Request

---

## 📄 License

Educational purposes - TryHackMe Advent of Cyber 2025

---

## 🙏 Acknowledgments

- **TryHackMe** for Advent of Cyber 2025
- **PortSwigger** for Burp Suite
- The web security community


---

**🎄 Happy Hacking! 🔐**

*In concurrent systems, timing is everything. Test under load, implement proper locking, and never trust that operations happen in order.*
