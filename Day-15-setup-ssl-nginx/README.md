# Day 15: Setup SSL for Nginx 

## Task Overview
The system admins team of xFusionCorp Industries needed to prepare **App Server 3 (`stapp03`)** for application deployment by installing **Nginx**, configuring **SSL/TLS** using a pre-existing self-signed certificate, and serving a custom index page — all accessible over HTTPS from the Jump host.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | `stapp03` (App Server 3) |
| Service | Nginx with SSL |
| Certificate Source | `/tmp/nautilus.crt` |
| Key Source | `/tmp/nautilus.key` |
| Certificate Destination | `/etc/certs/` |
| Index Content | `Welcome!` |
| Test Command | `curl -Ik https://<app-server-ip>/` from Jump host |

---

## 🛠️ Solution

### Step 1: SSH into App Server 3

```bash
ssh banner@stapp03
```

---

### Step 2: Install Nginx

```bash
sudo yum install nginx -y
```

---

### Step 3: Create Custom Index Page

Replace the default Nginx index page with the required content:

```bash
echo "Welcome!" | sudo tee /usr/share/nginx/html/index.html
```

---

### Step 4: Start Nginx & Test HTTP First

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

Quick HTTP test from Jump host:

```bash
curl http://stapp03
```

---

### Step 5: Move SSL Certificate & Key to Secure Location

```bash
sudo mkdir -p /etc/certs
sudo cp /tmp/nautilus.crt /etc/certs/
sudo cp /tmp/nautilus.key /etc/certs/

# Secure the key file permissions
sudo chmod 600 /etc/certs/nautilus.key
sudo chmod 644 /etc/certs/nautilus.crt
```

---

### Step 6: Configure Nginx for SSL

Edit the Nginx config:

```bash
sudo vi /etc/nginx/nginx.conf
```

**Inside the `server` block listening on port `80`**, add an HTTP → HTTPS redirect right after `server_name`:

```nginx
server {
    listen       80;
    server_name  _;

    # Add this line to redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}
```

**Uncomment and update the `server` block for port `443`**:

```nginx
server {
    listen       443 ssl;
    server_name  _;

    # SSL Certificate paths
    ssl_certificate     /etc/certs/nautilus.crt;
    ssl_certificate_key /etc/certs/nautilus.key;

    # Document root
    root   /usr/share/nginx/html;
    index  index.html index.htm;

    location / {
    }
}
```

---

### Step 7: Test Nginx Configuration

**Always test before restarting:**

```bash
sudo nginx -t
```

Expected output:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

### Step 8: Restart Nginx & Verify

```bash
sudo systemctl restart nginx
sudo systemctl status nginx
```

---

### Step 9: Final Test from Jump Host

```bash
curl -Ik https://stapp03/
# or using IP
curl -Ik https://<stapp03-ip>/
```

Expected output:

```
HTTP/1.1 200 OK
Server: nginx/1.x.x
Content-Type: text/html
...
```

---

## ✅ Verification Output

```
HTTP/2 200
server: nginx/1.20.1
content-type: text/html
...

Welcome!
```

---

## 🔄 Setup Flow

```
Install Nginx
     │
     ▼
Create index.html → "Welcome!"
     │
     ▼
Move certs to /etc/certs/
     │
     ▼
Edit nginx.conf:
  - Port 80  → redirect to HTTPS (301)
  - Port 443 → ssl_certificate + ssl_certificate_key
     │
     ▼
nginx -t  (test config)
     │
     ▼
systemctl restart nginx
     │
     ▼
curl -Ik https://stapp03/ ✅
```

---

## 💡 Key Concepts Learned

**Self-Signed vs CA-Signed Certificates:**
Self-signed certs encrypt traffic but aren't trusted by browsers (no chain of trust). The `-k` flag in `curl -Ik` skips certificate verification — needed for self-signed certs in testing.

**HTTP → HTTPS Redirect:**
`return 301 https://$host$request_uri;` in the port 80 block permanently redirects all HTTP traffic to HTTPS, ensuring users always get an encrypted connection.

**`nginx -t` — Always Test First:**
Never restart Nginx without running `nginx -t` first. A config error will bring down the service and make the server unreachable.

**Private Key Security:**
The `.key` file must have `600` permissions (owner read/write only). If it's world-readable, Nginx may still start but it's a serious security risk.

---

## 🔧 Useful Commands

| Command | Purpose |
|---------|---------|
| `nginx -t` | Test Nginx configuration syntax |
| `nginx -s reload` | Reload config without downtime |
| `systemctl status nginx` | Check Nginx service status |
| `curl -Ik https://host/` | Test HTTPS (skip cert verification) |
| `openssl x509 -in cert.crt -text` | Inspect certificate details |
| `chmod 600 /etc/certs/*.key` | Secure private key permissions |

---
