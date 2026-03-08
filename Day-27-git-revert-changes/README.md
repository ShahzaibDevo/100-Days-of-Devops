# Day 27: Git Revert Some Changes ↩️

## Task Overview
The Nautilus development team reported an issue with the **most recent commit** pushed to the `demo` repository on the Storage Server. The DevOps team was asked to safely undo the latest commit by using `git revert` — preserving the full commit history while rolling back the changes.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Repository Path | `/usr/src/kodekloudrepos/demo` |
| Action | Revert `HEAD` to previous commit |
| Revert Commit Message | `revert demo` (all lowercase) |
| Push | Yes — push the revert commit to origin |

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

---

### Step 3: Check Git Log to Understand Current State

```bash
git log --oneline
```

Example output:

```
f3a9c12 (HEAD -> master, origin/master) added index.html
a1b2c3d Initial commit
```

> The latest commit (`HEAD`) is what needs to be reverted back to `Initial commit`.

---

### Step 4: Revert the Latest Commit

```bash
git revert HEAD -m "revert demo"
```

> **Note:** The `-m "revert demo"` flag sets the commit message directly. If your Git version opens an editor instead, type `revert demo`, save, and close.

Expected output:

```
[master d4e5f67] revert demo
 1 file changed, 1 deletion(-)
```

---

### Step 5: Verify the Revert

```bash
git log --oneline
```

Expected:

```
d4e5f67 (HEAD -> master) revert demo
f3a9c12 added index.html
a1b2c3d Initial commit
```

> The revert creates a **new commit** — the bad commit is still in history but its changes are undone. ✅

---

### Step 6: Push the Changes to Remote

```bash
git push
```

---

## ✅ Verification

Check that the working directory reflects the state before the bad commit:

```bash
git show HEAD        # shows what the revert commit changed
git diff HEAD~1      # should show empty (reverted to previous state)
```

---

## 🔄 How Git Revert Works

```
Before Revert:
  A (Initial commit) ──► B (added index.html) ◄── HEAD

After git revert HEAD:
  A (Initial commit) ──► B (added index.html) ──► C (revert demo) ◄── HEAD

The changes from B are undone in C, but B is still in history.
History is preserved — safe for shared/public repositories.
```

---

## 💡 Key Concepts Learned

**`git revert` vs `git reset` — The Critical Difference:**

| Feature | `git revert` | `git reset` |
|---------|-------------|-------------|
| How it works | Creates a NEW undo commit | Moves HEAD pointer back |
| History | Preserved ✅ | Rewritten ❌ |
| Safe for shared repos | ✅ Yes | ❌ No (breaks others' history) |
| Recoverable | Always | Only before garbage collection |
| Use case | Public / team branches | Local / private branches only |

**Why `git revert` is the right choice here:**
The bad commit was already pushed to `origin` (shared). Using `git reset --hard` would rewrite history and force anyone who pulled the repo to face conflicts. `git revert` adds a new commit that undoes the changes — no history is lost, no one is disrupted.

**`git revert HEAD` explained:**
- `HEAD` refers to the latest commit on the current branch
- `git revert HEAD` creates a new commit that is the exact inverse of `HEAD`'s changes
- The `-m "message"` flag sets the commit message inline (skips editor)

---

## 🔧 Git Revert Command Reference

| Command | Purpose |
|---------|---------|
| `git revert HEAD` | Revert the latest commit |
| `git revert HEAD -m "msg"` | Revert with inline commit message |
| `git revert <hash>` | Revert a specific commit |
| `git revert HEAD~3..HEAD` | Revert last 3 commits |
| `git revert --no-commit HEAD` | Stage revert changes without committing |
| `git log --oneline` | View commit history |
| `git show HEAD` | Show details of latest commit |

---

## 🔑 Why This Matters

In production environments, bad commits get pushed to shared repositories all the time. Knowing the difference between `git revert` (safe, history-preserving) and `git reset` (destructive, history-rewriting) is critical. The golden rule: **never use `git reset` on commits that have already been pushed to a shared remote.** Always use `git revert` to keep history clean and protect your teammates' workflows.

---
