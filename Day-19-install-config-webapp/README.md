# Day 19: Install and Configure Web Application 🌐

## Task Overview
xFusionCorp Industries needed to host **two static websites** on **App Server 3 (`stapp03`)** in Stratos DC. The website backups were available on the Jump host and needed to be deployed under separate URL paths on a single Apache instance running on a custom port.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | `stapp03` (App Server 3) |
| Package | `httpd` |
| Apache Port | `6400` |
| Website 1 | `official` → `http://localhost:6400/official/` |
| Website 2 | `games` → `http://localhost:6400/games/` |
| Backup Source | `/home/thor/official` & `/home/thor/games` on Jump host |

---

## 🛠️ Solution

### Step 1: Install Apache on App Server 3

```bash
ssh banner@stapp03
sudo yum install -y httpd
```

---

### Step 2: Change Apache Port from 80 to 6400

Always back up the config before editing:

```bash
sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak
sudo sed -i 's/80/6400/g' /etc/httpd/conf/httpd.conf
```

Verify the port change:

```bash
grep "Listen" /etc/httpd/conf/httpd.conf
```

Expected:

```
Listen 6400
```

---

### Step 3: Start Apache

```bash
sudo systemctl enable --now httpd
sudo systemctl status httpd
```

---

### Step 4: Copy Website Backups from Jump Host to App Server

**Run on the Jump host:**

```bash
scp -r /home/thor/official banner@stapp03:/home/banner/
scp -r /home/thor/games banner@stapp03:/home/banner/
```

---

### Step 5: Deploy Websites to Apache Document Root

**Back on `stapp03`:**

```bash
sudo cp -r /home/banner/official /var/www/html/
sudo cp -r /home/banner/games /var/www/html/
```

Verify files are in place:

```bash
ls /var/www/html/
```

Expected:

```
games  official
```

---

### Step 6: Set Correct Permissions

```bash
sudo chmod -R 755 /var/www/html/official
sudo chmod -R 755 /var/www/html/games
```

---

### Step 7: Restart Apache & Verify

```bash
sudo systemctl restart httpd
```

Test both websites:

```bash
curl http://localhost:6400/official/
curl http://localhost:6400/games/
```

---

## ✅ Verification Output

```html
<!-- official -->
<!DOCTYPE html>
<html>
<body>
<h1>KodeKloud</h1>
<p>This is a sample page for our official website</p>
</body>
</html>

<!-- games -->
<!DOCTYPE html>
<html>
<body>
<h1>KodeKloud</h1>
<p>This is a sample page for our games website</p>
</body>
</html>
```

---

## 🔄 Deployment Flow

```
Jump Host
  /home/thor/official  ──scp──►  stapp03:/home/banner/official
  /home/thor/games     ──scp──►  stapp03:/home/banner/games
                                        │
                                        │ sudo cp
                                        ▼
                              /var/www/html/official/
                              /var/www/html/games/
                                        │
                                        ▼
                              Apache (port 6400)
                                        │
                          ┌─────────────┴─────────────┐
                          ▼                           ▼
              /official/  (site 1)         /games/  (site 2)
```

---

## 💡 Key Concepts Learned

**Subdirectory-based hosting:**
Both websites are served from the **same Apache instance and IP**, differentiated only by their URL path (`/official/` vs `/games/`). This is the simplest multi-site setup — no virtual hosts needed, just subdirectories under the document root `/var/www/html/`.

**`scp -r` for directory transfer:**
The `-r` flag copies directories recursively. Without it, `scp` only copies individual files and will fail on directories.

**`sed -i 's/80/6400/g'` — global port replacement:**
The `g` flag replaces **all** occurrences of `80` in the file. This updates both the `Listen 80` directive and any `VirtualHost *:80` blocks in one command.

**Apache Document Root structure:**
```
/var/www/html/           ← Main document root
    ├── official/        ← http://localhost:6400/official/
    │   └── index.html
    └── games/           ← http://localhost:6400/games/
        └── index.html
```

Apache automatically serves `index.html` when a directory URL is requested.

---

## 🔧 Useful Apache Commands

| Command | Purpose |
|---------|---------|
| `httpd -t` | Test Apache configuration syntax |
| `systemctl status httpd` | Check Apache service status |
| `apachectl -S` | Show virtual host settings |
| `grep "Listen" /etc/httpd/conf/httpd.conf` | Verify listening port |
| `ls /var/www/html/` | Check deployed files |
| `curl -I http://localhost:6400/` | Check HTTP response headers |

---

## 🔑 Why This Matters

Hosting multiple websites or applications under subdirectory paths is a common pattern for internal tools, staging environments, and microservices frontends. Understanding how Apache serves static content from subdirectories — and how to transfer files securely using `scp` — are foundational skills for deploying any web application in a real infrastructure environment.

---
