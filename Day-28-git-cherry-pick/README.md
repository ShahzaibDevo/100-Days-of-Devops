# Day 28: Git Cherry Pick 

## Task Overview
The Nautilus development team had work-in-progress on the `feature` branch but needed **one specific commit** — `Update info.txt` — merged into `master` without pulling the entire feature branch. This is a classic **cherry-pick** use case: selectively applying a single commit from one branch to another.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Repository Path | `/usr/src/kodekloudrepos/official` |
| Source Branch | `feature` |
| Target Branch | `master` |
| Commit to Pick | `Update info.txt` |
| Final Step | Push changes to origin |

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
cd /usr/src/kodekloudrepos/official
```

---

### Step 3: Check Available Branches

```bash
git branch
```

Expected:

```
* master
  feature
```

---

### Step 4: Find the Target Commit Hash on feature Branch

```bash
git log feature --oneline
```

Example output:

```
9f3a1b2 Add new UI components
c7d4e5f Update info.txt       ← this is the commit we need
a1b2c3d Initial commit
```

> Note the hash for the commit with message **`Update info.txt`** — e.g. `c7d4e5f`

---

### Step 5: Switch to Master Branch

```bash
git switch master
```

---

### Step 6: Cherry-Pick the Specific Commit

```bash
git cherry-pick c7d4e5f
```

Expected output:

```
[master e8f9a0b] Update info.txt
 Date: Mon Jan 01 10:00:00 2025 +0000
 1 file changed, 1 insertion(+)
```

---

### Step 7: Verify the Cherry-Pick

```bash
git log --oneline
```

Expected:

```
e8f9a0b (HEAD -> master) Update info.txt
a1b2c3d Initial commit
```

> The commit now exists on `master` with a **new hash** — same changes, new commit object. ✅

---

### Step 8: Push Changes to Remote

```bash
git push
```

---

## ✅ Visual: How Cherry-Pick Works

```
feature branch:
  A (Initial) ──► B (Update info.txt) ──► C (Add UI components) ──► ...
                        │
                        │  git cherry-pick B
                        ▼
master branch:
  A (Initial) ──────────────────────────► B' (Update info.txt)  ◄── HEAD

Note: B' is a new commit with same changes as B but a different hash.
The feature branch continues untouched with its full history.
```

---

## 💡 Key Concepts Learned

**What is Cherry-Pick?**
`git cherry-pick` takes the changes introduced by a specific commit and applies them as a **brand new commit** on the current branch. It copies the diff — not the commit itself — which is why the new commit gets a different hash.

**Cherry-Pick vs Merge vs Rebase:**

| Operation | What it does | Use case |
|-----------|-------------|----------|
| `git merge` | Combines entire branch history | Integrate complete features |
| `git rebase` | Replays all commits on new base | Linear history cleanup |
| `git cherry-pick` | Applies one specific commit | Selective commit transfer |

**When to use Cherry-Pick:**
- Applying a **bug fix** from `dev` to `production` without merging all dev changes
- **Backporting** a feature to an older release branch
- Pulling a single completed piece of work from a WIP branch
- Recovering accidentally committed changes from the wrong branch

**Why the new commit has a different hash:**
A Git commit hash is based on its content, author, timestamp, and **parent commit**. Since the cherry-picked commit has a different parent on `master` than on `feature`, it gets a new hash — even though the file changes are identical.

---

## 🔧 Git Cherry-Pick Command Reference

| Command | Purpose |
|---------|---------|
| `git cherry-pick <hash>` | Apply a single commit |
| `git cherry-pick hash1 hash2` | Apply multiple specific commits |
| `git cherry-pick start..end` | Apply a range of commits |
| `git cherry-pick --no-commit <hash>` | Stage changes without committing |
| `git cherry-pick --continue` | Continue after resolving conflicts |
| `git cherry-pick --abort` | Cancel the cherry-pick operation |
| `git log branch --oneline` | Find commit hash on another branch |

---

## 🔑 Why This Matters

Cherry-pick is an essential tool in release management and hotfix workflows. In real production environments, you often can't wait for an entire feature branch to be ready — a critical bug fix or a single important change needs to reach production immediately. `git cherry-pick` lets you do exactly that with surgical precision, without disturbing the rest of the codebase. It's a daily tool for DevOps engineers managing multiple release tracks.

---
