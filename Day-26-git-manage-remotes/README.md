# Day 26: Git Manage Remotes 🔗

## Task Overview
The xFusionCorp development team made changes on the Git server hosted on the **Storage Server** in Stratos DC. New Git remotes were added server-side, and the local repository needed to be updated accordingly — adding a new remote, committing a file, and pushing to that specific remote.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Repository Path | `/usr/src/kodekloudrepos/beta` |
| New Remote Name | `dev_beta` |
| New Remote URL | `/opt/xfusioncorp_beta.git` |
| File to Add | `/tmp/index.html` |
| Push Target | `master` branch → `dev_beta` remote |

---

## 🛠️ Solution

### Step 1: SSH into Storage Server

```bash
ssh natasha@ststor01
sudo su
```

---

### Step 2: Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos/beta
```

---

### Step 3: Check Current Remotes & Branch

```bash
git remote -v
git branch
```

Example current state:

```
origin  /opt/beta.git (fetch)
origin  /opt/beta.git (push)
* master
```

---

### Step 4: Add the New Remote

```bash
git remote add dev_beta /opt/xfusioncorp_beta.git
```

Verify the new remote was added:

```bash
git remote -v
```

Expected output:

```
dev_beta  /opt/xfusioncorp_beta.git (fetch)
dev_beta  /opt/xfusioncorp_beta.git (push)
origin    /opt/beta.git (fetch)
origin    /opt/beta.git (push)
```

---

### Step 5: Copy File & Commit to Master

```bash
cp /tmp/index.html .
git add .
git commit -m "added tmp file"
```

Expected:

```
[master abc1234] added tmp file
 1 file changed, 1 insertion(+)
```

---

### Step 6: Push Master to the New Remote

```bash
git push dev_beta master
```

Expected:

```
Counting objects: 3, done.
Writing objects: 100% (3/3), done.
To /opt/xfusioncorp_beta.git
 * [new branch]  master -> master
```

---

## ✅ Verification

Confirm both remotes are present and master is up to date:

```bash
git remote -v
git log --oneline
```

Check the new remote has received the push:

```bash
git ls-remote dev_beta
```

---

## 🔄 Workflow Diagram

```
Storage Server
│
├── /opt/beta.git              (origin — existing remote)
├── /opt/xfusioncorp_beta.git  (dev_beta — new remote)
│
└── /usr/src/kodekloudrepos/beta/   (local working repo)
         │
         │  git remote add dev_beta /opt/xfusioncorp_beta.git
         │  cp /tmp/index.html .
         │  git add . && git commit
         │
         │  git push dev_beta master
         ▼
    /opt/xfusioncorp_beta.git  ✅ (receives master branch)
```

---

## 💡 Key Concepts Learned

**Multiple Remotes:**
A single local Git repository can have **multiple remotes** — each pointing to a different server or path. This is common when you need to:
- Push to both a production server and a backup server
- Maintain both `origin` (your fork) and `upstream` (original repo)
- Deploy to different environments (staging vs production)

**`git push remote-name branch-name`:**
By default `git push` sends to `origin`. To push to a specific remote, explicitly name it:
```bash
git push dev_beta master    # push master to dev_beta
git push origin master      # push master to origin
```

**Local path as a remote URL:**
Git remotes don't have to be network URLs. A local filesystem path like `/opt/xfusioncorp_beta.git` works perfectly as a remote — useful for server-side setups where everything is on the same machine.

**Remote naming conventions:**

| Remote Name | Typical Use |
|-------------|-------------|
| `origin` | Default — your primary remote (clone source) |
| `upstream` | Original repo you forked from |
| `dev_beta` | Custom — specific environment or team remote |
| `backup` | Secondary remote for redundancy |

---

## 🔧 Git Remote Command Reference

| Command | Purpose |
|---------|---------|
| `git remote -v` | List all remotes with URLs |
| `git remote add name url` | Add a new remote |
| `git remote remove name` | Delete a remote |
| `git remote rename old new` | Rename a remote |
| `git remote set-url name url` | Update remote URL |
| `git push remote branch` | Push to a specific remote |
| `git push -u remote branch` | Push and set upstream tracking |
| `git ls-remote name` | List refs in a remote repo |
| `git fetch remote` | Download from specific remote |

---

## 🔑 Why This Matters

Managing multiple Git remotes is a critical skill in enterprise DevOps workflows. Teams often maintain separate remotes for development, staging, and production environments — pushing the same branch to different remotes to trigger different pipelines or deployments. Understanding how to add, configure, and push to named remotes gives you fine-grained control over where your code goes and when.

---
