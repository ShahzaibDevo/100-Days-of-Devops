# Day 24: Git Create Branches & Merge 

## Task Overview
The Nautilus development team needed a branching and merging workflow performed on the `demo` repository on the **Storage Server** in Stratos DC. The task involved creating a new branch, adding a file, committing it, merging back into `master`, and pushing both branches to the remote origin.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Repository Path | `/usr/src/kodekloudrepos/demo` |
| Remote Origin | `/opt/demo.git` |
| New Branch | `nautilus` (from `master`) |
| File to Add | `/tmp/index.html` |
| Final State | `nautilus` merged into `master`, both pushed |

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
cd /usr/src/kodekloudrepos/demo
```

Confirm you are on the `master` branch:

```bash
git branch
```

Expected:

```
* master
```

---

### Step 3: Create & Switch to New Branch

```bash
git checkout -b nautilus
```

Expected output:

```
Switched to a new branch 'nautilus'
```

---

### Step 4: Copy the File into the Repository

```bash
cp /tmp/index.html .
```

Verify the file is present:

```bash
ls
```

---

### Step 5: Stage & Commit the File

```bash
git add .
git commit -m "added tmp file"
```

Expected:

```
[nautilus abc1234] added tmp file
 1 file changed, 1 insertion(+)
```

---

### Step 6: Push the nautilus Branch to Remote

```bash
git push origin nautilus
```

---

### Step 7: Switch Back to Master & Merge

```bash
git switch master
git merge nautilus
```

Expected (fast-forward merge):

```
Updating abc1234..def5678
Fast-forward
 index.html | 1 +
 1 file changed, 1 insertion(+)
```

---

### Step 8: Push Master Branch to Remote

```bash
git push origin master
```

---

## ✅ Verification

Check both branches exist and commit history is consistent:

```bash
git log --oneline --all --graph
```

Expected:

```
* def5678 (HEAD -> master, origin/master, origin/nautilus, nautilus) added tmp file
* abc1234 Initial commit
```

Both branches point to the same commit after the fast-forward merge ✅

---

## 🔄 Complete Workflow Diagram

```
master:   A ──────────────────► A──B  (after merge + push)
               \                 ↑
nautilus:        └──► A──B ──► merge
                     (index.html added)

Steps:
1. git checkout -b nautilus    (branch from master)
2. cp /tmp/index.html .        (add file)
3. git add . && git commit     (commit on nautilus)
4. git push origin nautilus    (push nautilus)
5. git switch master           (switch back)
6. git merge nautilus          (fast-forward merge)
7. git push origin master      (push master)
```

---

## 💡 Key Concepts Learned

**`git checkout -b` vs `git switch -c`:**
Both create and switch to a new branch in one command. `git switch` is the newer, more intuitive command introduced in Git 2.23, while `checkout -b` is the classic approach. Both work identically for this purpose.

**Fast-Forward Merge:**
Since `nautilus` branched directly from `master` and `master` had no new commits in between, Git performed a **fast-forward merge** — it simply moved the `master` pointer forward to the `nautilus` commit. No merge commit was created, keeping the history linear and clean.

**Pushing both branches separately:**
`git push` alone only pushes the current branch. To push `nautilus` before switching back to `master`, explicitly specify: `git push origin nautilus`. Then after merging, push `master` as well.

**Branch from `master` workflow:**
```
master  →  git checkout -b feature  →  make changes  →  merge back  →  push both
```
This is the fundamental feature branch workflow used in almost every Git-based team.

---

## 🔧 Useful Git Branch Commands

| Command | Purpose |
|---------|---------|
| `git branch` | List all local branches |
| `git branch -a` | List local + remote branches |
| `git checkout -b name` | Create and switch to new branch |
| `git switch branch` | Switch to existing branch |
| `git merge branch` | Merge branch into current branch |
| `git push origin branch` | Push specific branch to remote |
| `git branch -d name` | Delete a merged branch |
| `git log --oneline --graph --all` | Visual branch history |

---

## 🔑 Why This Matters

Branching and merging is the heart of modern Git workflows — whether it's **GitFlow**, **trunk-based development**, or **feature branch workflows**. Creating isolated branches for each feature or fix, then merging them back to the mainline, keeps the codebase stable while enabling parallel development. This task covers the complete lifecycle: branch → develop → merge → push — a sequence every DevOps and developer performs daily.

---
