# Day 22: Clone Git Repository on Storage Server 📥

## Task Overview
The Nautilus application development team needed a working copy of an existing Git repository on the **Storage Server** in Stratos DC. The task was to clone the bare repository from a local path into a specific destination directory — without making any modifications to the repository.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Server | Storage Server (`ststor01`) |
| Source Repository | `/opt/games.git` (bare repo) |
| Destination | `/usr/src/kodekloudrepos` |
| Constraint | No modifications to the repository |

---

## 🛠️ Solution

### Step 1: SSH into Storage Server

```bash
ssh natasha@ststor01
```

---

### Step 2: Navigate to Destination Directory

```bash
cd /usr/src/kodekloudrepos
```

> If the directory doesn't exist, create it first:
> ```bash
> sudo mkdir -p /usr/src/kodekloudrepos
> cd /usr/src/kodekloudrepos
> ```

---

### Step 3: Clone the Repository

```bash
git clone /opt/games.git
```

Expected output:

```
Cloning into 'games'...
done.
```

---

### Step 4: Verify the Clone

```bash
ls /usr/src/kodekloudrepos/
```

Expected:

```
games
```

Check the cloned repo and its remote origin:

```bash
cd games
git log --oneline
git remote -v
```

Expected remote output:

```
origin  /opt/games.git (fetch)
origin  /opt/games.git (push)
```

---

## ✅ Verification Output

```
/usr/src/kodekloudrepos/
└── games/
    ├── .git/          ← full Git history
    └── (project files)
```

The clone automatically:
- Sets `origin` → `/opt/games.git`
- Checks out the default branch
- Copies the complete commit history

---

## 🔄 How Git Clone Works

```
/opt/games.git          (bare repo — source)
      │
      │  git clone /opt/games.git
      ▼
/usr/src/kodekloudrepos/games/    (working repo — destination)
      ├── .git/                   ← Git metadata + full history
      ├── remote: origin = /opt/games.git
      └── (checked-out working files)
```

---

## 💡 Key Concepts Learned

**Cloning from a local path:**
`git clone` works with local filesystem paths just like remote URLs. When the source is a bare repo (`.git` directory), Git copies all objects and refs and creates a working copy at the destination.

**What `git clone` does automatically:**
1. Creates the destination directory (`games/`)
2. Copies all Git objects (commits, trees, blobs)
3. Sets up `origin` remote pointing to the source
4. Checks out the default branch (usually `main` or `master`)
5. Sets up remote tracking branches

**Bare repo → Working repo:**
Cloning a bare repo (`.git` suffix, `bare = true`) produces a normal working repository — complete with a working directory and checked-out files. This is exactly what developers need to start working.

---

## 🔧 Common Git Clone Variants

| Command | Use Case |
|---------|----------|
| `git clone /path/repo.git` | Clone from local path |
| `git clone https://github.com/user/repo.git` | Clone over HTTPS |
| `git clone git@github.com:user/repo.git` | Clone over SSH |
| `git clone --depth 1 URL` | Shallow clone (latest commit only) |
| `git clone --branch dev URL` | Clone specific branch |
| `git clone --bare URL` | Clone as bare repo (no working dir) |
| `git clone URL /custom/path` | Clone into a specific directory |

---

## 🔑 Why This Matters

Cloning repositories is the entry point for every developer joining a project. Understanding that `git clone` works with local paths — not just remote URLs — is important for managing repositories on internal servers, storage systems, and air-gapped environments where internet access isn't available. This is a common pattern in enterprise DevOps workflows.

---
