# Day 12: Linux Network Services 🌐

## Task Overview
A monitoring alert reported that **Apache service is unreachable on port `3000`** on `stapp01` in Stratos DC. The root cause needed to be identified and fixed — whether it was the service itself, a port conflict, or a firewall issue — without compromising any security settings.

**Final test:** `curl http://stapp01:3000` from the Jump host must return a valid response.

---

## 📋 Requirements

| Detail | Value |
|--------|-------|
| Affected Server | `stapp01` |
| Expected Port | `3000` |
| Service | Apache (`httpd`) |
| Test Command | `curl http://stapp01:3000` from Jump host |

---

## 🔍 Root Cause Analysis

Two issues were found:

1. **Port conflict** — `sendmail` was already occupying port `3000`, preventing Apache from binding to it.
2. **Firewall block** — `iptables` had a `REJECT all` rule blocking inbound traffic on port `3000`.

---

## 🛠️ Solution

### Step 1: SSH into App Server & Check Apache Status

```bash
ssh tony@stapp01
sudo systemctl status httpd
```

Output showed Apache failed to start:

```
(98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:3000
no listening sockets available, shutting down
```

---

### Step 2: Check What's Using Port 3000

```bash
sudo netstat -tlnup
```

```
tcp   0   0 127.0.0.1:3000   0.0.0.0:*   LISTEN   430/sendmail: accep
```

> `sendmail` was occupying port `3000` — Apache couldn't bind to it.

---

### Step 3: Fix the Port Conflict (Change sendmail Port)

```bash
cd /etc/mail
sudo cp sendmail.mc sendmail.mc.bak      # backup first
sudo vi sendmail.mc
```

Find and update this line — change port `3000` to another value (e.g. `1234`):

```bash
# Before
DAEMON_OPTIONS(`Port=3000,Addr=127.0.0.1, Name=MTA')dnl

# After
DAEMON_OPTIONS(`Port=1234,Addr=127.0.0.1, Name=MTA')dnl
```

Restart sendmail:

```bash
sudo systemctl restart sendmail
```

---

### Step 4: Start Apache & Verify Port

```bash
sudo systemctl start httpd
sudo systemctl status httpd
sudo netstat -tlnup
```

Apache should now be listening on port `3000`.

---

### Step 5: Check Firewall Rules

Test from Jump host first:

```bash
telnet stapp01 3000
# Result: No route to host — firewall is blocking
```

Inspect iptables rules on `stapp01`:

```bash
sudo iptables -L -n
```

```
Chain INPUT (policy ACCEPT)
ACCEPT   all   0.0.0.0/0   0.0.0.0/0   state RELATED,ESTABLISHED
ACCEPT   icmp  0.0.0.0/0   0.0.0.0/0
ACCEPT   all   0.0.0.0/0   0.0.0.0/0
ACCEPT   tcp   0.0.0.0/0   0.0.0.0/0   state NEW tcp dpt:22
REJECT   all   0.0.0.0/0   0.0.0.0/0   reject-with icmp-host-prohibited
```

> The final `REJECT all` rule was blocking port `3000` traffic.

---

### Step 6: Fix the Firewall

Insert an ACCEPT rule for port `3000` **before** the REJECT rule (position 4):

```bash
sudo iptables -I INPUT 4 -p tcp --dport 3000 -j ACCEPT
```

Verify the updated rules:

```bash
sudo iptables -L -n
```

---

### Step 7: Verify Everything Works

From `stapp01`:

```bash
curl http://localhost:3000
```

From Jump host:

```bash
curl http://stapp01:3000
```

Expected: HTML response from Apache ✅

---

## ✅ Verification Output

```
HTTP/1.1 200 OK
...
<html><body><h1>It works!</h1></body></html>
```

---

## 💡 Key Concepts Learned

**Port Conflict Resolution:**
When two services try to bind to the same port, the second one fails. Always check `netstat -tlnup` or `ss -tlnup` to identify which process owns a port before starting a service.

**iptables Rule Order:**
Rules are processed **top to bottom** — first match wins. A blanket `REJECT all` at the bottom blocks everything not explicitly allowed above it. Inserting an `ACCEPT` rule at the correct position (before the REJECT) opens only the needed port.

**`-I` vs `-A` in iptables:**
- `-I INPUT 4` — **inserts** at position 4 (before the REJECT rule)
- `-A INPUT` — **appends** to the end (would be after REJECT, useless here)

---


