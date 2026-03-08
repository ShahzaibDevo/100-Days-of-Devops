# Day 32: Git Rebase 

## Task Overview
A developer on the Nautilus team was working on the `feature` branch while new commits were pushed to `master`. The developer wanted to bring those `master` changes into their `feature` branch — but **without a merge commit**, keeping the history linear and clean. The solution: `git rebase`.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Repository Path | `/usr/src/kodekloudrepos/games` |
| Working Branch | `feature` |
| Rebase Source | `master` |
| Constraint | No merge commits — linear history only |
| Push | `git push --force` required after rebase |

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
cd /usr/src/kodekloudrepos/games
```

---

### Step 3: Confirm You Are on the feature Branch

```bash
git branch
```

Expected:

```
* feature
  master
```

If not on `feature`:

```bash
git switch feature
```

---

### Step 4: Check Current State (Optional)

```bash
git log --oneline --all --graph
```

Example before rebase:

```
* f3a9c12 (HEAD -> feature) Feature work B
* a1b2c3d Feature work A
| * d4e5f67 (master) New master commit
| * c7d4e5f Master update
|/
* 848a423 Initial commit
```

---

### Step 5: Rebase feature onto master

```bash
git rebase master
```

Expected output:

```
Successfully rebased and updated refs/heads/feature.
```

---

### Step 6: Verify the Linear History

```bash
git log --oneline --all --graph
```

Expected after rebase:

```
* g8h9i0j (HEAD -> feature) Feature work B
* e6f7g8h Feature work A
* d4e5f67 (master) New master commit
* c7d4e5f Master update
* 848a423 Initial commit
```

> Feature commits are now replayed **on top of** the latest master — clean linear history, no merge commit! ✅

---

### Step 7: Push the Rebased Feature Branch

```bash
git push --force --set-upstream origin feature
```

> `--force` is required because rebase rewrites commit history (new hashes for feature commits).
> `--set-upstream` sets the remote tracking for the branch.

---

## ✅ Before vs After Rebase

```
BEFORE rebase:
                            master
                              │
  Initial ──► MasterUpdate ──► NewMasterCommit
        │
        └──► FeatureA ──► FeatureB  ◄── feature HEAD
             (branched from Initial)

AFTER git rebase master:
  Initial ──► MasterUpdate ──► NewMasterCommit ──► FeatureA' ──► FeatureB'
                                      │                               │
                                   master                          feature HEAD

Feature commits replayed on top of master — linear, clean, no merge commit.
```

---


## 🔑 Why This Matters

Rebasing is a fundamental skill for maintaining clean, readable Git history in professional projects. Many teams use rebase as part of their workflow to avoid cluttered merge commits, making `git log` and `git bisect` (bug hunting) much easier. Understanding when to rebase (personal feature branches) vs merge (shared/public branches) is a key mark of Git proficiency — and something every DevOps engineer working with CI/CD pipelines and PR workflows encounters daily.

---
