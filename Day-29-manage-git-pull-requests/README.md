# Day 29: Manage Git Pull Requests 

## Task Overview
The team decided that **no one should push directly to the `master` branch**. All changes must go through a **Pull Request (PR)** review process. Max had already pushed his story to a feature branch on Gitea and needed to raise a PR, assign a reviewer, and get it merged into `master` — the right way.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Developer | Max (`max` / `Max_pass123`) |
| Reviewer | Tom (`tom` / `Tom_pass123`) |
| Feature Branch | `story/fox-and-grapes` |
| Target Branch | `master` |
| PR Title | `Added fox-and-grapes story` |
| Platform | Gitea (self-hosted) |

---

## 🛠️ Solution

### Part 1: Explore the Repository (as Max)

SSH into the storage server as Max and inspect the repository:

```bash
ssh max@ststor01
# Password: Max_pass123

ls ~/                          # find the cloned repo
cd <repo-directory>

git log --oneline              # view commit history
git branch -a                  # confirm story/fox-and-grapes exists
```

Confirm Max's branch is already pushed to the remote:

```bash
git log --oneline origin/story/fox-and-grapes
```

---

### Part 2: Create the Pull Request (Gitea UI as Max)

**Step 1:** Click the **Gitea UI** button on the top bar.

**Step 2:** Login with Max's credentials:
- **Username:** `max`
- **Password:** `Max_pass123`

**Step 3:** Navigate to the repository and click **"New Pull Request"** or go to the **Pull Requests** tab.

**Step 4:** Configure the PR:

```
Source (compare):       story/fox-and-grapes
Destination (base):     master
Title:                  Added fox-and-grapes story
```

**Step 5:** Click **"Create Pull Request"**.

---

### Part 3: Assign Tom as Reviewer

**Step 6:** On the newly created PR page, look for the **Reviewers** section on the right sidebar.

**Step 7:** Click on **Reviewers** → search for `tom` → click to assign.

```
Reviewers
  [+ tom]  ← Add tom as reviewer
```

---

### Part 4: Review & Merge the PR (Gitea UI as Tom)

**Step 8:** Log out of Max's account and login as Tom:
- **Username:** `tom`
- **Password:** `Tom_pass123`

**Step 9:** Navigate to the Pull Request `Added fox-and-grapes story`.

**Step 10:** Review the changes — inspect the files diff.

**Step 11:** Approve and **Merge** the PR.

```
[Merge Pull Request] ← Click to merge story/fox-and-grapes into master
```

---

## ✅ Result

```
story/fox-and-grapes  (Max's branch)
        │
        │  PR: "Added fox-and-grapes story"
        │  Reviewer: tom ✅ Approved
        │
        ▼
    master  (merged) 🎉
```

Max's fox-and-grapes story is now part of the `master` branch — reviewed, approved, and merged the right way!

---

## 🔄 Complete Pull Request Workflow

```
1. Developer works on feature branch
   git checkout -b story/fox-and-grapes
   git add . && git commit -m "Add fox and grapes story"
   git push origin story/fox-and-grapes

2. Open Pull Request (Gitea UI)
   Source:      story/fox-and-grapes
   Destination: master
   Title:       Added fox-and-grapes story

3. Assign Reviewer
   Reviewers → Add tom

4. Code Review (as Tom)
   - Read the diff
   - Leave comments if needed
   - Approve the PR

5. Merge PR
   master now contains the story ✅

6. (Optional) Delete feature branch after merge
```

---

## 💡 Key Concepts Learned

**Why no direct pushes to master?**
The `master` (or `main`) branch is the **source of truth** — it should always contain stable, reviewed, approved code. Allowing direct pushes bypasses code review, breaks quality gates, and can introduce bugs or security issues directly into production.

**Pull Request vs Merge:**

| Feature | Direct Merge | Pull Request |
|---------|-------------|--------------|
| Code review | ❌ None | ✅ Required |
| Discussion | ❌ None | ✅ Threaded comments |
| Audit trail | ❌ Limited | ✅ Full history |
| Quality gate | ❌ None | ✅ Approvals required |
| Team visibility | ❌ Low | ✅ Everyone can see |

**Branch protection rules:**
In Gitea (and GitHub/GitLab), you can enforce PR reviews by setting **branch protection rules** on `master`:
- Require X number of approvals before merge
- Dismiss approvals when new commits are pushed
- Require status checks to pass

**Gitea PR components:**
- **Source branch** — where your changes live
- **Target branch** — where changes will be merged
- **Reviewers** — team members assigned to review
- **Labels/Milestones** — organization and tracking
- **Diff view** — side-by-side file change comparison

---

## 🔧 Git Commands Related to PRs

```bash
# Check remote branches
git branch -r

# Push feature branch to remote
git push origin story/fox-and-grapes

# Fetch all remote branches
git fetch --all

# Check PR merge locally (after merge)
git pull origin master
git log --oneline --graph --all
```

---

## 🔑 Why This Matters

The Pull Request workflow is the **backbone of modern software development**. Every major company — from startups to FAANG — uses PRs to enforce code review, maintain quality, and build a collaborative engineering culture. Understanding how to create PRs, assign reviewers, and manage the approval process is a non-negotiable skill for any DevOps engineer working in team environments. This workflow applies identically across GitHub, GitLab, Gitea, and Bitbucket.

---
