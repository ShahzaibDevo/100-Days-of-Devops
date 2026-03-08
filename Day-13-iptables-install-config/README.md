# Day 13: IPtables Installation And Configuration 

## Task Overview
The security team raised a concern that **Apache's port `6200` is open to everyone** on all app servers in Stratos DC. To add a security layer, we needed to install `iptables`, restrict port `6200` to only allow traffic from the **Load Balancer (LBR)** host, and ensure the rules **persist after reboot**.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Action | Install `iptables` on all app servers |
| Protected Port | `6200` (Apache) |
| Allowed Source | LBR host only (`172.16.238.14`) |
| Block | All other incoming traffic on port `6200` |
| Persistence | Rules must survive system reboot |

---

## 🛠️ Solution

### Step 1: Login to Each App Server

```bash
# App Server 1
ssh tony@stapp01

# App Server 2
ssh steve@stapp02

# App Server 3
ssh banner@stapp03
```

Run the following steps on **each** app server:

---

### Step 2: Install iptables & iptables-services

```bash
sudo yum install -y iptables iptables-services
```

> `iptables-services` is required for persistence — it provides the `service iptables save` command and auto-loads rules on reboot.

---

### Step 3: Add ACCEPT Rule for LBR Host Only

Allow incoming traffic on port `6200` **only** from the LBR host (`172.16.238.14`):

```bash
sudo iptables -A INPUT -p tcp --dport 6200 -s 172.16.238.14 -j ACCEPT
```

---

### Step 4: Block Port 6200 for Everyone Else

Reject all other incoming traffic on port `6200`:

```bash
sudo iptables -A INPUT -p tcp --dport 6200 -j REJECT
```

---

### Step 5: Save Rules for Persistence

Make the rules survive a system reboot:

```bash
sudo service iptables save
```

> This writes rules to `/etc/sysconfig/iptables` which is auto-loaded on boot by `iptables-services`.

---

### Step 6: Enable iptables Service on Boot

```bash
sudo systemctl enable iptables
sudo systemctl start iptables
```

---

### Step 7: Verify the Rules

```bash
sudo iptables -L -n --line-numbers
```

Expected output for INPUT chain:

```
Chain INPUT (policy ACCEPT)
num  target   prot opt source           destination
1    ACCEPT   tcp  --  172.16.238.14    0.0.0.0/0    tcp dpt:6200
2    REJECT   tcp  --  0.0.0.0/0        0.0.0.0/0    tcp dpt:6200  reject-with icmp-port-unreachable
```

---

## ✅ How It Works

```
Incoming request on port 6200
         │
         ▼
  Source = 172.16.238.14 (LBR)?
         │
    YES  │  NO
    ─────┼─────
         │       │
      ACCEPT   REJECT
   (LBR allowed) (everyone else blocked)
```

**Rule order matters** — iptables processes rules **top to bottom, first match wins**:
- Rule 1: If source is LBR IP → `ACCEPT`
- Rule 2: All other traffic on port 6200 → `REJECT`

---

## 💡 Key Concepts Learned

**`-A` (Append) vs `-I` (Insert):**
- `-A` adds the rule at the **bottom** of the chain
- `-I` inserts at a specific position (use when order is critical)
- Here, ACCEPT must come before REJECT — appending in correct order achieves this

**`REJECT` vs `DROP`:**
- `REJECT` sends back an error message (ICMP port unreachable) — cleaner for internal networks
- `DROP` silently discards packets — better against external attackers

**Persistence with `iptables-services`:**
- Without `iptables-services`, rules are lost on reboot
- `service iptables save` writes to `/etc/sysconfig/iptables`
- The service reads this file automatically at boot

---

## 🔧 Useful iptables Commands

| Command | Purpose |
|---------|---------|
| `iptables -L -n --line-numbers` | List rules with line numbers |
| `iptables -S` | Show rules as commands |
| `iptables -F` | Flush (delete) all rules |
| `iptables -D INPUT 2` | Delete rule number 2 from INPUT |
| `iptables-save` | Print all current rules |
| `service iptables save` | Persist rules to disk |

---
