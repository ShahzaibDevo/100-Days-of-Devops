# Day 14: Linux Process Troubleshooting 

## Task Overview
The monitoring system reported **Apache service unavailability** on one of the app servers in Stratos DC. The goal was to identify the faulty host, fix the issue, and ensure Apache is **up and running on port `3004`** across all app servers.

---

## 📋 Requirements

| Detail | Value |
|--------|-------|
| Affected Service | Apache (`httpd`) |
| Required Port | `3004` |
| Scope | All app servers (`stapp01`, `stapp02`, `stapp03`) |
| Test Command | `curl http://stapp01:3004` from Jump host |

---

## 🔍 Root Cause Analysis

Apache failed to start because **`sendmail` was already occupying port `3004`**, preventing `httpd` from binding to it.

```
(98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:3004
no listening sockets available, shutting down
```

---

## 🛠️ Solution

### Step 1: SSH into App Server & Check Apache Status

```bash
ssh tony@stapp01   # or steve@stapp02 / banner@stapp03
sudo systemctl status httpd
```

Apache showed `failed` status with error:

```
Active: failed (Result: exit-code)
httpd[1003]: (98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:3004
httpd[1003]: no listening sockets available, shutting down
httpd[1003]: AH00015: Unable to open logs
```

---

### Step 2: Identify the Port Conflict

```bash
sudo netstat -tlnup
```

```
Proto  Local Address      State   PID/Program
tcp    127.0.0.1:3004     LISTEN  680/sendmail: accep
tcp    0.0.0.0:22         LISTEN  443/sshd
```

> `sendmail` is holding port `3004` — Apache cannot start until this is resolved.

---

### Step 3: Fix sendmail Port Conflict

Back up and edit the sendmail config:

```bash
cd /etc/mail
sudo cp sendmail.mc sendmail.mc.bak
sudo vi sendmail.mc
```

Find the line with port `3004` and change it to an unused port (e.g. `1234`):

```bash
# Before
DAEMON_OPTIONS(`Port=3004,Addr=127.0.0.1, Name=MTA')dnl

# After
DAEMON_OPTIONS(`Port=1234,Addr=127.0.0.1, Name=MTA')dnl
```

Restart sendmail to apply the change:

```bash
sudo systemctl restart sendmail
```

---

### Step 4: Start Apache & Verify

```bash
sudo systemctl start httpd
sudo systemctl status httpd
```

Expected output:

```
Active: active (running)
```

Confirm Apache is now on port `3004`:

```bash
sudo netstat -tlnup | grep 3004
```

```
tcp   0.0.0.0:3004   LISTEN   1103/httpd
```

---

### Step 5: Test from Jump Host

```bash
curl http://stapp01:3004
```

Expected: Apache default page HTML response ✅

---

## ✅ Verification Output

```
HTTP/1.1 200 OK
...
<html><body><h1>It works!</h1></body></html>
```

---

## 🔄 Troubleshooting Flow

```
Apache fails to start
        │
        ▼
  Check systemctl status httpd
        │
        ▼
  Error: Address already in use (port 3004)
        │
        ▼
  netstat -tlnup → sendmail using port 3004
        │
        ▼
  Edit /etc/mail/sendmail.mc → change port
        │
        ▼
  Restart sendmail → Start httpd
        │
        ▼
  curl http://stapp01:3004 ✅
```

---

## 💡 Key Concepts Learned

**Reading systemctl error output:**
The key clue was `(98)Address already in use` — error code 98 on Linux always means a port binding conflict. This immediately points to `netstat` as the next diagnostic step.

**Why sendmail uses port 3004:**
sendmail's `DAEMON_OPTIONS` in `sendmail.mc` defines which port it listens on. This is a compiled macro file — after editing, the service must be restarted to pick up changes.

**Process troubleshooting order:**
1. Check service status (`systemctl status`)
2. Read the error message carefully
3. Check port usage (`netstat -tlnup`)
4. Identify the conflicting process
5. Resolve the conflict (change port or stop service)
6. Start the target service
7. Verify with `curl` or `telnet`

---

## 🔧 Troubleshooting Toolkit

| Command | Purpose |
|---------|---------|
| `systemctl status httpd` | Check Apache service status |
| `journalctl -u httpd -f` | Live Apache service logs |
| `netstat -tlnup` | Show all listening ports with PID |
| `ss -tlnup` | Modern alternative to netstat |
| `lsof -i :3004` | Show which process uses port 3004 |
| `httpd -t` | Test Apache config syntax |
| `/var/log/httpd/error_log` | Apache error log file |

---

