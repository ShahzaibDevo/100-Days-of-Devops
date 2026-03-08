# Day 33: Resolve Git Merge Conflicts 

## Task Overview
Sarah and Max were both working on the `story-blog` repository. Max tried to push his new changes but encountered a **merge conflict** because Sarah had also pushed changes to the same file. The task was to fetch the latest changes, resolve the conflict in `story-index.txt` (ensuring all 4 story titles are present and fixing a typo), and push the clean result to origin.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| User | `max` / `Max_pass123` |
| Repository | `/home/max/story-blog` |
| Conflicted File | `story-index.txt` |
| Fix 1 | All 4 story titles must be present |
| Fix 2 | Typo: `Mooose` → `Mouse` in "The Lion and the Mouse" |
| Final Step | Push resolved changes to origin |

---

## 🛠️ Solution

### Step 1: SSH into Storage Server as Max

```bash
ssh max@ststor01
# Password: Max_pass123
```

---

### Step 2: Navigate to the Repository

```bash
cd /home/max/story-blog
```

Check current status:

```bash
git status
git log --oneline
```

---

### Step 3: Fetch Remote Changes

```bash
git fetch origin
```

This downloads Sarah's latest commits without modifying the working directory.

---

### Step 4: Attempt Rebase (triggers conflict)

```bash
git rebase origin/main
```

Git will pause and report a conflict:

```
CONFLICT (content): Merge conflict in story-index.txt
error: could not apply abc1234... Max's commit
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
```

---

### Step 5: Inspect & Resolve the Conflict

Open the conflicted file:

```bash
vi story-index.txt
```

You will see conflict markers like this:

```
<<<<<<< HEAD (Sarah's version on origin)
1. The Lion and the Mooose
2. The Lazy Donkey
=======
1. The Lion and the Mooose
3. The Three Little Pigs
4. Puss in Boots
>>>>>>> abc1234 (Max's changes)
```

**Resolve by keeping ALL 4 stories AND fixing the typo:**

```
1. The Lion and the Mouse
2. The Lazy Donkey
3. The Three Little Pigs
4. Puss in Boots
```

> - Remove all `<<<<<<<`, `=======`, `>>>>>>>` conflict markers
> - Fix `Mooose` → `Mouse`
> - Ensure all 4 story titles are present

---

### Step 6: Stage the Resolved File

```bash
git add story-index.txt
```

---

### Step 7: Continue the Rebase

```bash
git rebase --continue
```

Git will prompt for a commit message — save and close the editor, or it will use the existing commit message automatically.

---

### Step 8: Push the Resolved Changes

```bash
git push
```

> If push is rejected due to rebase rewriting history:
> ```bash
> git push --force
> ```

---

## ✅ Final state of story-index.txt

```
1. The Lion and the Mouse
2. The Lazy Donkey
3. The Three Little Pigs
4. Puss in Boots
```

All 4 titles present ✅ | Typo fixed ✅

---

## 🔄 Conflict Resolution Flow

```
git fetch origin          (download Sarah's changes)
        │
        ▼
git rebase origin/main    (replay Max's commits on top)
        │
        ▼  ⚠️ CONFLICT in story-index.txt
        │
        ▼
vi story-index.txt        (manually resolve — keep all 4, fix typo)
        │
        ▼
git add story-index.txt   (mark as resolved)
        │
        ▼
git rebase --continue     (complete the rebase)
        │
        ▼
git push                  (push clean history to origin) ✅
```

---

## 💡 Key Concepts Learned

**What causes a merge conflict?**
When two people edit the **same lines** of the same file on different branches, Git cannot automatically decide which version to keep. It marks the file with conflict markers and pauses for a human decision.

**Understanding conflict markers:**

```
<<<<<<< HEAD
(current branch changes — what's on origin/main from Sarah)
=======
(incoming changes — Max's commits being rebased)
>>>>>>> commit-hash
```

## 🔑 Why This Matters

Merge conflicts are an unavoidable part of working in any team. The ability to calmly read conflict markers, understand what each side is proposing, and craft the correct resolution is one of the most important practical Git skills. The rebase workflow keeps history clean while the core skill — editing conflict markers and staging the resolution — applies equally to `git merge`, `git rebase`, and `git cherry-pick` conflicts.

---
