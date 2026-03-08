# Day 30: Git Hard Reset 

## Task Overview
The Nautilus development team had been running test commits on the `ecommerce` repository on the **Storage Server**. They wanted to completely clean the commit history and point `HEAD` back to the `add data.txt file` commit — leaving only two commits in the repository's history. This required a `git reset --hard` followed by a force push.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Repository Path | `/usr/src/kodekloudrepos/ecommerce` |
| Target Commit | `add data.txt file` |
| Final History | Only 2 commits: `initial commit` + `add data.txt file` |
| Push Method | `git push --force` (history rewrite) |

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

### Step 3: View Current Commit History

```bash
git log --oneline
```

Current state — 12 commits total:

```
81e08fc (HEAD -> master, origin/master) Test Commit10
686cb01 Test Commit9
f0ff119 Test Commit8
a8404ed Test Commit7
a9fccf0 Test Commit6
2554746 Test Commit5
461c467 Test Commit4
287d981 Test Commit3
a63283d Test Commit2
5479bcb Test Commit1
90b2925 add data.txt file      ← reset target
848a423 initial commit
```

> Note the hash of `add data.txt file` — `90b2925`

---

### Step 4: Reset HEAD to Target Commit

```bash
git reset --hard 90b2925
```

Expected output:

```
HEAD is now at 90b2925 add data.txt file
```

Verify the branch is now behind origin:

```bash
git status
```

```
On branch master
Your branch is behind 'origin/master' by 10 commits, and can be fast-forwarded.
nothing to commit, working tree clean
```

---

### Step 5: Force Push to Overwrite Remote History

```bash
git push --force
```

Expected output:

```
Total 0 (delta 0), reused 0 (delta 0)
To /opt/ecommerce.git
 + 81e08fc...90b2925 master -> master (forced update)
```

---

### Step 6: Verify Final State

```bash
git log --oneline
```

Expected — only 2 commits remain:

```
90b2925 (HEAD -> master, origin/master) add data.txt file
848a423 initial commit
```

✅ All 10 test commits have been removed from history.

---

## 🔄 Before vs After

```
BEFORE reset:
  848a423 initial commit
  90b2925 add data.txt file
  5479bcb Test Commit1
  a63283d Test Commit2
  287d981 Test Commit3
  461c467 Test Commit4
  2554746 Test Commit5
  a9fccf0 Test Commit6
  a8404ed Test Commit7
  f0ff119 Test Commit8
  686cb01 Test Commit9
  81e08fc Test Commit10  ◄── HEAD

AFTER reset:
  848a423 initial commit
  90b2925 add data.txt file  ◄── HEAD (10 commits erased)
```

---

## 💡 Key Concepts Learned

**Three Types of Git Reset:**

| Reset Type | HEAD | Staging Area | Working Directory | Use Case |
|------------|------|-------------|-------------------|----------|
| `--soft` | Moved | Unchanged | Unchanged | Keep changes staged |
| `--mixed` *(default)* | Moved | Cleared | Unchanged | Keep changes unstaged |
| `--hard` | Moved | Cleared | Cleared | Completely discard changes |

**`git reset --hard` vs `git revert`:**

| Feature | `git reset --hard` | `git revert` |
|---------|--------------------|--------------|
| History | Rewritten (destroyed) | Preserved |
| New commit | ❌ No | ✅ Yes (undo commit) |
| Safe for shared repos | ❌ Never | ✅ Always |
| Push method needed | `--force` | Normal push |
| Recoverable | Only via `git reflog` | Always |

**Why `git push --force` is needed:**
After a hard reset, the local branch is **behind** the remote (the remote still has all 10 test commits). A normal `git push` would be rejected because it would erase remote history. `--force` tells Git to overwrite the remote with the local state regardless.

**Emergency Recovery with `git reflog`:**
Even after `--hard` reset, commits aren't immediately deleted — they're just unreachable. You can recover them within ~90 days:

```bash
git reflog                    # shows all recent HEAD movements
git reset --hard HEAD@{1}     # restore to state before reset
```

---

## 🔧 Git Reset Command Reference

| Command | Purpose |
|---------|---------|
| `git log --oneline` | View commit history with hashes |
| `git reset --hard <hash>` | Reset to specific commit, discard all changes |
| `git reset --soft HEAD~1` | Undo last commit, keep changes staged |
| `git reset --mixed HEAD~1` | Undo last commit, keep changes unstaged |
| `git push --force` | Force push (overwrite remote history) |
| `git push --force-with-lease` | Safer force push (checks for others' changes) |
| `git reflog` | View history of HEAD movements |
| `git reset --hard HEAD@{n}` | Recover from reflog |

---

## 🔑 Why This Matters

Understanding `git reset --hard` and `git push --force` is critical — both for using them correctly in appropriate scenarios (test repos, personal branches) and for knowing when **not** to use them (shared branches, production repos). A forced history rewrite on a shared repository breaks everyone else's local copy and can cause irreversible data loss. This is why `git revert` exists — for the cases where you need to undo changes safely. Knowing both tools and when to reach for each one is a mark of Git maturity.

---
