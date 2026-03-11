# Day 93: Using Ansible Conditionals 🔀

## Task Overview
The Nautilus DevOps team wanted to use **Ansible's `when` conditionals** to copy different files to different app servers — all from a single playbook targeting all hosts. Each server receives a specific file with correct ownership and permissions.

---

## 📋 Requirements

| Server | File | Owner | Permissions |
|--------|------|-------|-------------|
| `stapp01` | `blog.txt` | `tony` | `0644` |
| `stapp02` | `story.txt` | `steve` | `0644` |
| `stapp03` | `media.txt` | `banner` | `0644` |

- Source files located at `/usr/src/data/` on jump host
- Destination: `/opt/data/` on each app server
- Playbook must use `hosts: all` with `when` conditionals

---

## 🛠️ Solution

### Step 1: Create the Playbook

```bash
cat > ~/ansible/playbook.yml << 'EOF'
---
- name: Copy files to app servers using conditionals
  hosts: all
  become: true
  tasks:

    - name: Copy blog.txt to App Server 1
      copy:
        src: /usr/src/data/blog.txt
        dest: /opt/data/blog.txt
        owner: tony
        group: tony
        mode: '0644'
      when: inventory_hostname == "stapp01"

    - name: Copy story.txt to App Server 2
      copy:
        src: /usr/src/data/story.txt
        dest: /opt/data/story.txt
        owner: steve
        group: steve
        mode: '0644'
      when: inventory_hostname == "stapp02"

    - name: Copy media.txt to App Server 3
      copy:
        src: /usr/src/data/media.txt
        dest: /opt/data/media.txt
        owner: banner
        group: banner
        mode: '0644'
      when: inventory_hostname == "stapp03"
EOF
```

### Step 2: Run the Playbook

```bash
cd ~/ansible
ansible-playbook -i inventory playbook.yml
```

### Step 3: Verify

```bash
ansible all -i inventory -a "ls -la /opt/data/" -b
```

---

## 💡 Key Concepts Learned

**How `when` conditionals work:**
Every task runs on all hosts by default. The `when` condition checks a variable at runtime — if it evaluates to `false`, the task is **skipped** on that host. This is what produces the `skipping` messages in the output.

```
TASK [Copy blog.txt to App Server 1]
skipping: [stapp02]   ← when: false
skipping: [stapp03]   ← when: false
changed:  [stapp01]   ← when: true ✅
```

**`inventory_hostname` vs `ansible_nodename`:**

| Variable | Value | Notes |
|----------|-------|-------|
| `inventory_hostname` | `stapp01` | Name from inventory file — no fact gathering needed |
| `ansible_nodename` | System FQDN from OS | Requires gathered facts |

`inventory_hostname` is simpler and more reliable for this use case.

**Why `hosts: all` with `when` instead of separate plays:**
Using one play with `hosts: all` + `when` is cleaner than writing three separate plays. It also demonstrates Ansible's conditional capability — the intended learning objective of this task.

---

## 🔑 Why This Matters

`when` conditionals are one of Ansible's most used features in real infrastructure. They allow a single playbook to handle different OS families, different environments (dev/staging/prod), or different server roles — without duplicating tasks. Mastering conditionals is essential for writing flexible, reusable Ansible automation.

---
