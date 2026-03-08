# Day 20: Configure Nginx + PHP-FPM Using Unix Socket 🔌

## Task Overview
The Nautilus development team needed a **PHP-based application stack** deployed on **App Server 1 (`stapp01`)**. The task involved installing Nginx and PHP-FPM 8.2, configuring them to communicate via a **Unix socket**, and serving PHP content on a custom port.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | `stapp01` (App Server 1) |
| Web Server | Nginx on port `8093` |
| Document Root | `/var/www/html` |
| PHP Version | `8.2` (via PHP-FPM) |
| Unix Socket | `/var/run/php-fpm/default.sock` |
| Test Command | `curl http://stapp01:8093/index.php` from Jump host |

---

## 🏗️ Architecture Overview

```
   Jump Host
       │  HTTP Request :8093
       ▼
   Nginx (port 8093)
       │  FastCGI via Unix Socket
       │  /var/run/php-fpm/default.sock
       ▼
   PHP-FPM 8.2
       │  Executes PHP
       ▼
   /var/www/html/index.php
```

**Unix Socket vs TCP Socket:**
- Unix socket → faster, local only, more secure (filesystem permissions)
- TCP socket → network-capable but slower for local communication

---

## 🛠️ Solution

### Step 1: SSH into App Server 1

```bash
ssh tony@stapp01
```

---

### Step 2: Update Packages & Install Nginx + PHP-FPM 8.2

```bash
sudo dnf update -y
sudo dnf install nginx -y
sudo dnf module install php:8.2 -y
```

Verify PHP-FPM installed:

```bash
php-fpm --version
```

---

### Step 3: Configure PHP-FPM Unix Socket

Create the socket directory if it doesn't exist:

```bash
sudo mkdir -p /var/run/php-fpm
```

Edit the PHP-FPM pool config:

```bash
sudo vi /etc/php-fpm.d/www.conf
```

Find the `listen` directive and update it:

```ini
; Before
listen = /run/php-fpm/www.sock

; After
listen = /var/run/php-fpm/default.sock
```

---

### Step 4: Configure Nginx Port

Edit the main Nginx config:

```bash
sudo vi /etc/nginx/nginx.conf
```

Change port `80` to `8093`:

```nginx
server {
    listen       8093;
    listen       [::]:8093;
    server_name  _;
    root         /var/www/html;
    ...
}
```

---

### Step 5: Connect Nginx to PHP-FPM via Unix Socket

Edit the PHP FastCGI pass config:

```bash
sudo vi /etc/nginx/default.d/php.conf
```

Update the `fastcgi_pass` directive to point to the Unix socket:

```nginx
# Before
fastcgi_pass php-fpm;

# After
fastcgi_pass unix:/var/run/php-fpm/default.sock;
```

---

### Step 6: Create a Test PHP Page

```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/index.php
```

---

### Step 7: Enable & Start Services

```bash
sudo systemctl enable --now php-fpm
sudo systemctl enable --now nginx

# Verify both are running
sudo systemctl status php-fpm
sudo systemctl status nginx
```

---

### Step 8: Test Config & Verify

Test Nginx config before any restart:

```bash
sudo nginx -t
```

Expected:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Verify socket was created:

```bash
ls -la /var/run/php-fpm/default.sock
```

Expected:

```
srw-rw---- 1 nginx nginx ... /var/run/php-fpm/default.sock
```

---

### Step 9: Final Test from Jump Host

```bash
curl http://stapp01:8093/index.php
```

Expected: PHP info page HTML output ✅

---

## ✅ Key Config File Summary

| File | Change Made |
|------|-------------|
| `/etc/php-fpm.d/www.conf` | `listen = /var/run/php-fpm/default.sock` |
| `/etc/nginx/nginx.conf` | Port `80` → `8093` |
| `/etc/nginx/default.d/php.conf` | `fastcgi_pass unix:/var/run/php-fpm/default.sock;` |

---

## 💡 Key Concepts Learned

**What is PHP-FPM?**
PHP-FPM (FastCGI Process Manager) is a separate PHP process manager that runs independently from the web server. Nginx doesn't execute PHP natively — it forwards `.php` requests to PHP-FPM via the FastCGI protocol, gets the output, and sends it back to the client.

**Unix Socket vs TCP Socket:**
- `fastcgi_pass 127.0.0.1:9000;` — TCP socket (slower, network overhead even on localhost)
- `fastcgi_pass unix:/var/run/php-fpm/default.sock;` — Unix socket (faster, uses filesystem IPC, no network stack)

**Why `mkdir -p /var/run/php-fpm`?**
`/var/run` is a tmpfs (RAM-based) directory — it's cleared on every reboot. The subdirectory must be created before PHP-FPM starts, otherwise it can't create its socket file.

**`dnf module install php:8.2`:**
Modern RHEL/CentOS systems use DNF modules to manage multiple PHP versions side-by-side. The `:8.2` suffix selects the specific stream (version) to install.

---

## 🔧 Useful Commands

| Command | Purpose |
|---------|---------|
| `nginx -t` | Test Nginx config syntax |
| `php-fpm --version` | Check PHP-FPM version |
| `php-fpm -t` | Test PHP-FPM config |
| `ls -la /var/run/php-fpm/` | Verify socket file exists |
| `systemctl status php-fpm` | Check PHP-FPM service |
| `journalctl -u php-fpm -f` | Live PHP-FPM logs |
| `journalctl -u nginx -f` | Live Nginx logs |

---

## 🔑 Why This Matters

Nginx + PHP-FPM is the industry standard PHP deployment stack, replacing the older Apache + mod_php approach. It offers better performance, process isolation, and the ability to run multiple PHP versions simultaneously. Unix sockets further optimize the setup by eliminating TCP overhead for local communication — a pattern used in high-traffic production environments worldwide.

---
