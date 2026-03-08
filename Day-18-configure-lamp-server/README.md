# Day 18: Configure LAMP Server (LAMP Stack) 

## Task Overview
xFusionCorp Industries is deploying a **WordPress website** on Stratos DC infrastructure. The storage server already has a shared directory `/var/www/html` mounted on all app hosts. The task was to set up a full **LAMP stack** — configuring Apache with PHP on all app servers and MariaDB on the database server.

---

## 📋 Requirements

| Component | Detail |
|-----------|--------|
| App Servers | `stapp01`, `stapp02`, `stapp03` |
| Packages | `httpd`, `php`, `php-mysqli` |
| Apache Port | `3003` |
| DB Server | `stdb01` |
| DB Package | `mariadb-server` |
| Database | `kodekloud_db2` |
| DB User | `kodekloud_cap` |
| DB Password | `your-pass` |
| DB Access | Remote (`%`) — all hosts |

---

## 🏗️ LAMP Stack Overview

```
   Client Browser
        │  HTTP Request
        ▼
   Apache (httpd)         ← Port 3003
        │  PHP Request
        ▼
   PHP + php-mysqli        ← Processes app logic
        │  SQL Query
        ▼
   MariaDB (stdb01)        ← kodekloud_db2
```

**L** — Linux | **A** — Apache | **M** — MariaDB | **P** — PHP

---

## 🛠️ Solution

### Part 1: Configure All App Servers

> Repeat these steps on `stapp01`, `stapp02`, and `stapp03`

#### Step 1: SSH into App Server

```bash
ssh tony@stapp01      # repeat for steve@stapp02, banner@stapp03
```

#### Step 2: Install Apache, PHP & MySQL Extension

```bash
sudo yum install -y httpd php php-mysqli
```

#### Step 3: Change Apache Port from 80 to 3003

Always back up config before editing:

```bash
sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak
sudo sed -i 's/\<80\>/3003/g' /etc/httpd/conf/httpd.conf
```

Verify the port change:

```bash
grep "Listen" /etc/httpd/conf/httpd.conf
```

Expected:

```
Listen 3003
```

#### Step 4: Enable & Start Apache

```bash
sudo systemctl enable --now httpd
sudo systemctl status httpd
```

---

### Part 2: Configure Database Server

#### Step 1: SSH into DB Server

```bash
ssh peter@stdb01
```

#### Step 2: Install & Start MariaDB

```bash
sudo yum install -y mariadb-server
sudo systemctl enable --now mariadb
sudo systemctl status mariadb | grep "running"
```

#### Step 3: Create Database, User & Grant Permissions

```bash
# Create the database
mysql -u root -e "CREATE DATABASE kodekloud_db2;"

# Create the user with remote access (% = any host)
mysql -u root -e "CREATE USER 'kodekloud_cap'@'%' IDENTIFIED BY 'your-pass';"

# Grant full privileges on the database
mysql -u root -e "GRANT ALL ON kodekloud_db2.* TO 'kodekloud_cap'@'%';"

# Apply privilege changes
mysql -u root -e "FLUSH PRIVILEGES;"
```

> `'%'` in the host field allows connections from **any remote host** — required so app servers can connect to the DB server.

---

### Part 3: Verify Everything

Check Apache is listening on port 3003 (from app server):

```bash
sudo ss -tlnup | grep httpd
```

Expected:

```
tcp  LISTEN  0.0.0.0:3003  users:(("httpd",...))
```

Check MariaDB user and database (from DB server):

```bash
mysql -u root -e "SHOW DATABASES;" | grep kodekloud_db2
mysql -u root -e "SELECT User, Host FROM mysql.user;" | grep kodekloud_cap
```

Test remote DB connection from an app server:

```bash
mysql -u kodekloud_cap -p'your-pass' -h stdb01 -e "SHOW DATABASES;"
```

---

## ✅ Verification Output

Clicking the **App** button should display:

```
App is able to connect to the database using user kodekloud_cap
```

---

## 💡 Key Concepts Learned

**`php-mysqli` extension:**
The `php-mysqli` package is what allows PHP scripts to connect to MySQL/MariaDB databases. Without it, any PHP app trying to use `mysqli_connect()` will fail silently or throw a connection error.

**`sed -i 's/\<80\>/3003/g'` — Word boundary replacement:**
The `\<` and `\>` anchors ensure only the standalone number `80` is replaced — not `8080` or `80s`. This is safer than a plain `s/80/3003/g` which could corrupt other config values.

**`'user'@'%'` vs `'user'@'localhost'`:**
- `'localhost'` — only allows connections from the same machine
- `'%'` — allows connections from any remote host (required for app→DB communication across servers)

**`FLUSH PRIVILEGES`:**
Forces MariaDB to reload the grant tables immediately. Without this, new permissions might not take effect until the service restarts.

---

## 🔧 Useful MariaDB / MySQL Commands

| Command | Purpose |
|---------|---------|
| `SHOW DATABASES;` | List all databases |
| `SHOW TABLES;` | List tables in current DB |
| `SELECT User, Host FROM mysql.user;` | List all users |
| `SHOW GRANTS FOR 'user'@'%';` | Show user permissions |
| `FLUSH PRIVILEGES;` | Reload grant tables |
| `DROP DATABASE dbname;` | Delete a database |
| `DROP USER 'user'@'%';` | Delete a user |

---

## 🔑 Why This Matters

The LAMP stack powers a huge portion of the web — WordPress, Drupal, Joomla, and many custom apps all run on it. Understanding how to configure each layer (Apache port, PHP extensions, MariaDB users with remote access) and how they connect together is fundamental knowledge for any DevOps, SysAdmin, or backend developer. This task also reinforces the principle of separating application and database tiers across different servers — a standard production architecture pattern.

---
