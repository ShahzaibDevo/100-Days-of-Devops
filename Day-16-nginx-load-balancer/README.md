# Day 16: Install and Configure Nginx as a Load Balancer ⚖️

## Task Overview
Due to increasing traffic causing performance degradation, the Nautilus production team decided to deploy their application on a **high availability stack**. The final step was configuring the **LBR (Load Balancer) server** with Nginx to distribute traffic across all three app servers in Stratos DC.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| LBR Server | `stlb01` |
| Backend Servers | `stapp01`, `stapp02`, `stapp03` |
| Apache Port (App Servers) | `5001` (do not change) |
| Config File | `/etc/nginx/nginx.conf` (main file only) |
| Load Balancing Method | Round-robin (default) |

---

## 🛠️ Solution

### Step 1: Verify Apache is Running on All App Servers

SSH into each app server and confirm `httpd` is up and note the port:

```bash
# On each app server (stapp01, stapp02, stapp03)
sudo ss -tlnup | grep httpd
```

Expected output:

```
tcp  LISTEN  0.0.0.0:5001  users:(("httpd",pid=1680,fd=3))
```

> Apache is running on port `5001` — do **not** change this.

---

### Step 2: Install Nginx on LBR Server

```bash
ssh loki@stlb01

sudo yum install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

### Step 3: Configure Nginx as Load Balancer

Edit the main Nginx config file:

```bash
sudo vi /etc/nginx/nginx.conf
```

**Add the `upstream` block** inside the `http {}` section, just before the `server` block:

```nginx
upstream stapp {
    server stapp01:5001;
    server stapp02:5001;
    server stapp03:5001;
}
```

**Update the `location /` block** inside `server { listen 80; }` to proxy traffic to the upstream:

```nginx
location / {
    proxy_pass http://stapp;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    proxy_connect_timeout 5s;
    proxy_read_timeout 60s;
}
```

---

### Step 4: Full nginx.conf Reference

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;

    # Load balancer upstream pool
    upstream stapp {
        server stapp01:5001;
        server stapp02:5001;
        server stapp03:5001;
    }

    server {
        listen       80;
        listen       [::]:80;
        server_name  _;

        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html { }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html { }

        location / {
            proxy_pass http://stapp;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            proxy_connect_timeout 5s;
            proxy_read_timeout 60s;
        }
    }
}
```

---

### Step 5: Test Config & Restart Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl status nginx
```

Expected from `nginx -t`:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

### Step 6: Verify Load Balancer is Working

```bash
# From jump host - hit the LBR multiple times to see round-robin in action
curl http://stlb01
curl http://stlb01
curl http://stlb01
```

---

## ✅ Architecture Overview

```
                  ┌─────────────┐
   Client ──────► │  Nginx LBR  │  (stlb01:80)
                  └──────┬──────┘
                         │  Round-Robin
            ┌────────────┼────────────┐
            ▼            ▼            ▼
      stapp01:5001  stapp02:5001  stapp03:5001
       (Apache)      (Apache)      (Apache)
```

---

## 💡 Key Concepts Learned

**`upstream` block:**
Defines the pool of backend servers. Nginx distributes requests across them using **round-robin by default** — each new request goes to the next server in the list, cycling evenly.

**`proxy_pass`:**
Forwards incoming requests from the LBR to the upstream pool. The value `http://stapp` references the upstream block named `stapp`.

**Proxy Headers — why they matter:**
- `X-Real-IP` — passes the client's actual IP to backend servers (otherwise they'd only see the LBR's IP)
- `X-Forwarded-For` — builds a chain of all proxy IPs for audit/logging
- `Host` — preserves the original request hostname

**`proxy_connect_timeout` vs `proxy_read_timeout`:**
- `connect_timeout` — how long to wait when establishing connection to backend (5s)
- `read_timeout` — how long to wait for the backend to send a response (60s)

---

## 🔧 Load Balancing Methods in Nginx

| Method | Config | Use Case |
|--------|--------|----------|
| Round-robin | *(default)* | Equal load distribution |
| Least connections | `least_conn;` | Uneven request durations |
| IP Hash | `ip_hash;` | Session persistence (sticky) |
| Weighted | `server s1:80 weight=3;` | Servers with different capacity |
| Backup | `server s1:80 backup;` | Failover only |

---

## 🔑 Why This Matters

A single app server is a **single point of failure**. Load balancing across multiple servers provides high availability, horizontal scalability, and zero-downtime deployments. Nginx is one of the most widely used tools for this — lightweight, performant, and highly configurable. This setup mirrors real-world production architectures used by companies of all sizes.

---
