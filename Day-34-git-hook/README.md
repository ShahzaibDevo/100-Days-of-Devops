# Day 34: Git Hook — Auto Release Tagging 

## Task Overview
The Nautilus development team wanted to **automate release tagging** on their `demo` repository. Every time changes are pushed to the `master` branch, a `post-update` hook should automatically create a release tag with today's date (e.g. `release-2023-06-15`). The task also required merging the `feature` branch into `master` before setting up and testing the hook.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Repository | `/usr/src/kodekloudrepos/demo` |
| Bare Repo (hook location) | `/opt/demo.git/hooks/` |
| Task 1 | Merge `feature` → `master` |
| Task 2 | Create `post-update` hook |
| Hook Action | Auto-create release tag `release-YYYY-MM-DD` on push |
| Final Step | Push changes & verify tag |

---

## 🛠️ Solution

### Step 1: SSH into Storage Server

```bash
ssh natasha@ststor01
sudo -i
```

---

### Step 2: Navigate to the Cloned Repository

```bash
cd /usr/src/kodekloudrepos/demo
```

---

### Step 3: Switch to Master & Merge feature Branch

```bash
git branch              # confirm current branch
git switch master
git merge feature
```

Expected:

```
Updating abc1234..def5678
Fast-forward
 (feature files merged)
```

---

### Step 4: Create the post-update Hook

Copy the sample hook as a starting point:

```bash
cp /opt/demo.git/hooks/post-update.sample /opt/demo.git/hooks/post-update
chmod +x /opt/demo.git/hooks/post-update
```

---

### Step 5: Write the Hook Script

```bash
vi /opt/demo.git/hooks/post-update
```

Replace or append with the following content:

```bash
#!/bin/sh

# Auto release tagging hook
day=$(date +"%Y-%m-%d")
TAG="release-$day"
git tag -a $TAG -m "Released at: $day"

echo "Release tag created: $TAG"
```

Save and confirm the hook is executable:

```bash
ls -la /opt/demo.git/hooks/post-update
cat /opt/demo.git/hooks/post-update
```

---

### Step 6: Push Changes to Trigger the Hook

```bash
git push
```

The `post-update` hook will fire automatically on the bare repo after receiving the push, creating the release tag.

Expected output (from hook):

```
Release tag created: release-2023-06-15
```

---

### Step 7: Verify the Tag Was Created

```bash
git fetch --tags
git tag
```

Expected:

```
release-2023-06-15
```

Check tag details:

```bash
git show release-2023-06-15
```

---

## ✅ How the Hook Works

```
Developer runs: git push
        │
        ▼
Changes received by /opt/demo.git (bare repo)
        │
        ▼
post-update hook fires automatically
        │
        ▼
Script runs:
  day=$(date +"%Y-%m-%d")          → "2023-06-15"
  TAG="release-$day"               → "release-2023-06-15"
  git tag -a $TAG -m "Released at: $day"  → creates annotated tag
        │
        ▼
Release tag release-2023-06-15 created ✅
```

---




## 🔑 Why This Matters

Git hooks are the foundation of **CI/CD automation at the Git level** — before tools like Jenkins, GitHub Actions, or GitLab CI even get involved. `post-update` hooks are used in production to trigger deployments, send Slack notifications, create release artifacts, and update changelogs automatically. Understanding how to write and place server-side hooks is an essential DevOps skill for managing self-hosted Git servers and building lightweight automation pipelines.

---

