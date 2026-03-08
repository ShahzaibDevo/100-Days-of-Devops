# Day 17: Install and Configure PostgreSQL 🐘

## Task Overview
The Nautilus application development team is deploying a new application that requires a **PostgreSQL database**. The database server is already installed — the task was to create a database user, set a password, create a database, and grant full permissions to that user.

> ⚠️ **Important:** Do NOT restart the PostgreSQL service during this task.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Nautilus DB Server |
| Database User | `kodekloud_aim` |
| User Password | `your-password` |
| Database Name | `kodekloud_db6` |
| Permissions | Full privileges on `kodekloud_db6` |

---

## 🛠️ Solution

### Step 1: SSH into Database Server

```bash
ssh peter@stdb01
```

---

### Step 2: Switch to postgres System User

```bash
sudo -i -u postgres
```

> All PostgreSQL admin commands must be run as the `postgres` system user.

---

### Option A — Run Commands Directly from Terminal (Recommended)

Execute all commands without entering the psql shell:

```bash
# Create the database
psql -c "CREATE DATABASE kodekloud_db6;"

# Create the user/role with login and password
psql -c "CREATE ROLE kodekloud_aim LOGIN PASSWORD 'your-password';"

# Grant full privileges on the database to the user
psql -c "GRANT ALL PRIVILEGES ON DATABASE kodekloud_db6 TO kodekloud_aim;"

# Transfer database ownership to the user
psql -c "ALTER DATABASE kodekloud_db6 OWNER TO kodekloud_aim;"
```

---

### Option B — Interactive psql Shell

Login to the PostgreSQL interactive shell:

```bash
sudo psql -U postgres
```

Then run these SQL commands:

```sql
CREATE DATABASE kodekloud_db6;
CREATE ROLE kodekloud_aim LOGIN PASSWORD 'your-password';
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db6 TO kodekloud_aim;
ALTER DATABASE kodekloud_db6 OWNER TO kodekloud_aim;
\q
```

---

### Step 3: Verify the Setup

Check the database was created:

```bash
psql -c "\l" | grep kodekloud_db6
```

Check the user/role was created:

```bash
psql -c "\du" | grep kodekloud_aim
```

Connect as the new user to verify access:

```bash
psql -U kodekloud_aim -d kodekloud_db6 -c "\conninfo"
```

Expected output:

```
You are connected to database "kodekloud_db6" as user "kodekloud_aim"
```

---

## ✅ Verification Output

```
List of databases
      Name       |    Owner     
-----------------+--------------
 kodekloud_db6   | kodekloud_aim

List of roles
   Role name    | Attributes
----------------+------------
 kodekloud_aim  | 
 postgres       | Superuser, Create role...
```

---

## 🔄 Command Breakdown

```
sudo -i -u postgres
        │
        ▼
CREATE DATABASE kodekloud_db6
        │
        ▼
CREATE ROLE kodekloud_aim LOGIN PASSWORD '...'
        │
        ▼
GRANT ALL PRIVILEGES ON DATABASE → kodekloud_aim
        │
        ▼
ALTER DATABASE kodekloud_db6 OWNER TO kodekloud_aim
        │
        ▼
Verify with \l and \du ✅
```

---

## 💡 Key Concepts Learned

**Roles vs Users in PostgreSQL:**
PostgreSQL uses **roles** as a unified concept for both users and groups. A role with the `LOGIN` attribute functions as a user (can connect to the database). `CREATE USER` is actually an alias for `CREATE ROLE ... LOGIN`.

**`GRANT ALL PRIVILEGES` vs `ALTER ... OWNER`:**
- `GRANT ALL PRIVILEGES` — gives the role permission to use the database
- `ALTER DATABASE ... OWNER TO` — makes the role the owner, giving full control including the ability to drop the database and manage other users' access

**Why `sudo -i -u postgres`?**
PostgreSQL uses **peer authentication** by default for local connections — it checks that the OS user matches the database user. Since the superuser role is `postgres`, you must be the `postgres` OS user to run admin commands without a password.

**`psql -c` flag:**
Runs a single SQL command from the shell without entering the interactive psql prompt — cleaner for scripting and automation.

---

## 🔧 Useful psql Commands

| Command | Purpose |
|---------|---------|
| `\l` | List all databases |
| `\du` | List all roles/users |
| `\c dbname` | Connect to a database |
| `\dt` | List tables in current database |
| `\conninfo` | Show current connection info |
| `\q` | Quit psql shell |
| `\dp` | Show table access privileges |

---

## 🔑 Why This Matters

Database user management is a critical part of application security. Creating dedicated database users per application (rather than using the superuser) follows the **principle of least privilege** — if the app is compromised, the attacker only has access to that one database. This is standard practice in every production environment and is required for compliance with security frameworks like SOC 2 and ISO 27001.

---
