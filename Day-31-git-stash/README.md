# Day 31: Git Stash 

## Task Overview
A developer on the Nautilus team had previously stashed some **in-progress changes** in the `ecommerce` repository on the Storage Server. The task was to locate the correct stash entry (`stash@{1}`), restore it to the working directory, commit the changes, and push to the remote origin.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Repository Path | `/usr/src/kodekloudrepos/ecommerce` |
| Stash to Restore | `stash@{1}` |
| Action | Apply stash → commit → push to origin |

---

## 🛠️ Solution

### Step 1: SSH into Storage Server

```bash
ssh natasha@ststor01
sudo -i
```

---

### Step 2: Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos/ecommerce
```

---

### Step 3: List All Stashed Changes

```bash
git stash list
```

Output:

```
stash@{0}: WIP on master: 7fe985d initial commit
stash@{1}: WIP on master: 7fe985d initial commit
```

> Two stashes exist. We need to restore `stash@{1}` specifically.

---

### Step 4: Inspect the Stash Before Applying (Optional but Recommended)

```bash
git stash show -p stash@{1}
```

This shows the exact file changes stored in `stash@{1}` so you know what you're restoring.

---

### Step 5: Apply the Specific Stash

```bash
git stash apply stash@{1}
```

Expected output:

```
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
        modified: some-file.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

> `apply` restores the stash but **keeps it in the stash list** (unlike `pop` which removes it).

---

### Step 6: Stage, Commit & Push

```bash
git add .
git commit -m "Restored stash files"
git push
```

---

## ✅ Verification

Check the commit was pushed successfully:

```bash
git log --oneline
```

Expected:

```
abc1234 (HEAD -> master, origin/master) Restored stash files
7fe985d initial commit
```

Verify stash list is unchanged (stash still exists after `apply`):

```bash
git stash list
```

---

## 🔄 Git Stash Workflow Diagram

```
Developer working on changes
        │
        │  git stash push -m "WIP: feature work"
        ▼
Changes saved to stash stack:
  stash@{0}: latest stash
  stash@{1}: older stash   ← we restore this one
        │
        │  git stash apply stash@{1}
        ▼
Changes restored to working directory
        │
        │  git add . && git commit && git push
        ▼
Changes committed to master ✅
```

---



## 🔑 Why This Matters

`git stash` is one of the most practical day-to-day Git tools. In real development, you're often interrupted mid-task — a production bug needs fixing, a colleague needs a quick review, or you need to test something on a clean branch. Stash lets you save your work instantly without making a messy "WIP" commit, switch contexts, and come back later to pick up exactly where you left off. Understanding how to manage multiple stashes by index is essential when you've stashed changes across multiple sessions.

---

