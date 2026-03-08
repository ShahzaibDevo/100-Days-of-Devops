# Day 23: Fork a Git Repository 🍴

## Task Overview
A new developer **Jon** joined the Nautilus project team and needed to start working on an existing project. To follow best practices, Jon had to **fork** the repository `sarah/story-blog` into his own Gitea account before making any changes — a standard open-source and team collaboration workflow.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Platform | Gitea (self-hosted Git server) |
| Login User | `jon` |
| Password | `Jon_pass123` |
| Source Repository | `sarah/story-blog` |
| Fork Destination | `jon/story-blog` |

---

## 🛠️ Solution

### Step 1: Access Gitea UI

Click the **Gitea UI** button on the top bar to open the Gitea web interface.

---

### Step 2: Login as Jon

Enter the credentials on the login page:

- **Username:** `jon`
- **Password:** `Jon_pass123`

---

### Step 3: Navigate to the Source Repository

From the dashboard, find and click on `sarah/story-blog` in the right sidebar or search for it using the Explore tab.

---

### Step 4: Fork the Repository

On the `sarah/story-blog` repository page, click the **Fork** button in the top-right corner.

```
sarah/story-blog
[Watch] [Star] [Fork ▼]  ← Click here
```

In the fork dialog, confirm the destination owner as **jon** and click **Fork Repository**.

---

### Step 5: Verify the Fork

After forking, you will be redirected to `jon/story-blog`. Confirm:

- URL shows `jon/story-blog`
- Page shows **"forked from sarah/story-blog"**

---

## ✅ Result

```
sarah/story-blog  (original)
       │
       │  Fork
       ▼
jon/story-blog    (jon's personal copy)
  "forked from sarah/story-blog"
```

Jon now has full ownership of his fork and can make changes freely without affecting the original repository.

---

## 🔄 Complete Fork-Based Collaboration Workflow

```
1. Fork (Gitea UI)
   sarah/story-blog → jon/story-blog

2. Clone (local machine)
   git clone http://gitea-server/jon/story-blog

3. Create Feature Branch
   git checkout -b feature/new-chapter

4. Make Changes & Commit
   git add .
   git commit -m "Add new chapter"

5. Push to Fork
   git push origin feature/new-chapter

6. Open Pull Request
   jon/story-blog → sarah/story-blog (PR)

7. Review & Merge
   sarah reviews and merges the PR
```

---

## 💡 Key Concepts Learned

**Fork vs Clone:**

| Feature | Fork | Clone |
|---------|------|-------|
| Where it lives | Server-side (Gitea/GitHub) | Local machine |
| Ownership | You own the copy | Just a local download |
| Connection to original | Maintained (for PRs) | Via `remote` config |
| Purpose | Contribute to others' projects | Work locally |
| Created via | Web UI | `git clone` command |

**Why Fork instead of cloning directly?**
If you clone `sarah/story-blog` directly and make changes, you can't push them back (no write access). By forking first, you get your own server-side copy with full push access, while still maintaining a link to the original for submitting pull requests.

**Gitea vs GitHub/GitLab:**
Gitea is a lightweight, **self-hosted** Git service. Companies use it when they need GitHub-like features (web UI, PRs, issues, wikis) but want to keep code on their own infrastructure for security or compliance reasons.

---

## 🔧 Post-Fork Git Commands

```bash
# Clone your fork locally
git clone http://gitea-server/jon/story-blog
cd story-blog

# Add original repo as upstream (to sync future changes)
git remote add upstream http://gitea-server/sarah/story-blog

# Verify remotes
git remote -v
# origin   → jon/story-blog   (your fork)
# upstream → sarah/story-blog (original)

# Sync your fork with the original
git fetch upstream
git merge upstream/main
```

---
