

# Day 90: Managing ACLs Using Ansible 🔐

## Task Overview
Configure Access Control Lists (ACLs) on multiple app servers in **Stratos DC** using Ansible. Files must be owned by `root`, but app-specific users/groups need specific permissions — all automated via a single playbook.

---

## 📋 Requirements

| Server | File | Entity | Type | Permissions |
|--------|------|--------|------|-------------|
| stapp01 | `/opt/finance/blog.txt` | `tony` | group | `r` |
| stapp02 | `/opt/finance/story.txt` | `steve` | user | `rw` |
| stapp03 | `/opt/finance/media.txt` | `banner` | group | `rw` |

---

## 🛠️ Solution

### Step 1: Create the playbook

```bash
cat > /home/thor/ansible/playbook.yml << 'EOF'
---
- name: Manage ACLs on App Server 1 (stapp01)
  hosts: stapp01
  become: true
  tasks:
    - name: Create empty file blog.txt on app server 1
      file:
        path: /opt/finance/blog.txt
        state: touch
        owner: root
        group: root

    - name: Set ACL - give group tony read permission on blog.txt
      acl:
        path: /opt/finance/blog.txt
        entity: tony
        etype: group
        permissions: r
        state: present

- name: Manage ACLs on App Server 2 (stapp02)
  hosts: stapp02
  become: true
  tasks:
    - name: Create empty file story.txt on app server 2
      file:
        path: /opt/finance/story.txt
        state: touch
        owner: root
        group: root

    - name: Set ACL - give user steve read+write permission on story.txt
      acl:
        path: /opt/finance/story.txt
        entity: steve
        etype: user
        permissions: rw
        state: present

- name: Manage ACLs on App Server 3 (stapp03)
  hosts: stapp03
  become: true
  tasks:
    - name: Create empty file media.txt on app server 3
      file:
        path: /opt/finance/media.txt
        state: touch
        owner: root
        group: root

    - name: Set ACL - give group banner read+write permission on media.txt
      acl:
        path: /opt/finance/media.txt
        entity: banner
        etype: group
        permissions: rw
        state: present
EOF
```

### Step 2: Run the playbook

```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```

---


## 💡 Key Concepts Learned

**What is ACL?**
Access Control Lists (ACLs) provide fine-grained permission control beyond the standard Unix `owner/group/others` model. They allow you to grant specific permissions to individual users or groups on a file without changing its ownership.

**Ansible `acl` module parameters:**
- `path` — target file
- `entity` — the user or group name
- `etype` — either `user` or `group`
- `permissions` — `r`, `w`, `x`, or combinations like `rw`
- `state: present` — ensures the ACL entry exists

**Ansible `file` module with `state: touch`:**
- Creates an empty file if it doesn't exist
- Does not overwrite if it already exists
- Combined with `owner/group: root` ensures root ownership

---

