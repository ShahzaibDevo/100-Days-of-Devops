# Day 21: Set Up Git Repository on Storage Server 

## Task Overview
The Nautilus development team requested a **central Git repository** for a new application project. The task was to install Git on the Storage Server in Stratos DC and initialize a **bare repository** at a specific path — the standard setup for a shared, team-accessible Git server.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Package | `git` (via yum) |
| Repository Path | `/opt/demo.git` |
| Repository Type | Bare repository |

---

## 🛠️ Solution

### Step 1: SSH into Storage Server

```bash
ssh natasha@ststor01
```

---

### Step 2: Install Git

```bash
sudo yum update -y
sudo yum install -y git
```

Verify installation:

```bash
git --version
```

---

### Step 3: Create the Bare Repository

```bash
sudo git init --bare /opt/demo.git
```

Expected output:

```
Initialized empty Git repository in /opt/demo.git/
```

---

### Step 4: Verify the Repository Structure

```bash
ls /opt/demo.git/
```

Expected output:

```
HEAD  branches  config  description  hooks  info  objects  refs
```

---

## ✅ Verification

Check the repository config to confirm it's bare:

```bash
cat /opt/demo.git/config
```

Expected:

```ini
[core]
    repositoryformatversion = 0
    filemode = true
    bare = true
```

> `bare = true` confirms this is a bare repository ✅

---

## 🔄 How Developers Use This Repository

Once the bare repo is set up, developers can clone and push to it:

```bash
# Clone from a developer machine
git clone natasha@ststor01:/opt/demo.git

# Or add it as a remote to an existing project
git remote add origin natasha@ststor01:/opt/demo.git
git push origin main
```

---

## 💡 Key Concepts Learned

**What is a Bare Repository?**
A bare repository contains **only the Git data** (history, commits, branches, objects) with no working directory or checked-out files. Created with `git init --bare`, it's the standard way to set up a **central shared repository** on a server.

**Working Repo vs Bare Repo:**

| Feature | Working Repo | Bare Repo |
|---------|-------------|-----------|
| Working directory | ✅ Yes | ❌ No |
| Checked-out files | ✅ Yes | ❌ No |
| Can `git push` to it | ❌ No (causes errors) | ✅ Yes |
| Used by | Developers locally | Servers / central hubs |
| Created with | `git init` | `git init --bare` |

**Why `.git` suffix for bare repos?**
By convention, bare repositories use a `.git` suffix (e.g., `demo.git`) to signal that the directory is a Git repo without a working tree. GitHub, GitLab, and Gitea all follow this convention internally.

**Why you can't push to a non-bare repo:**
Pushing to a working repository would overwrite the checked-out files of whoever is working there — causing conflicts. Bare repos have no working directory, so pushes are always safe.

---

## 🔧 Useful Git Server Commands

| Command | Purpose |
|---------|---------|
| `git init --bare /path/repo.git` | Create a bare repository |
| `git clone user@host:/path/repo.git` | Clone from a remote server |
| `cat repo.git/config` | Verify bare = true |
| `ls repo.git/` | Inspect bare repo structure |
| `git remote -v` | Show remote URLs |
| `git remote add origin URL` | Add a remote to local repo |

---

## 🔑 Why This Matters

Setting up a central bare Git repository is the foundation of any team-based development workflow. Before tools like GitHub or GitLab existed, teams ran their own Git servers exactly this way — and many enterprises still do for security and compliance reasons. Understanding how bare repositories work also demystifies what services like GitHub actually store under the hood.

---
