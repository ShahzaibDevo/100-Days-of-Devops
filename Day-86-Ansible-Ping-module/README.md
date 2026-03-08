#  Day 86 of #100DaysOfDevOps — Ansible Ping Module Usage


##  Task Overview

The Nautilus DevOps team needed to set up **password-less SSH** between the Ansible controller (Jump Host) and managed nodes (App Servers) in **Stratos DC**, then validate connectivity using the **Ansible ping module**.

**Requirements:**
- Jump Host = Ansible Controller (running as `thor` user)
- Use inventory file at `/home/thor/ansible/inventory`
- Test Ansible ping from Jump Host → **App Server 3 (stapp03)**

---

##  Infrastructure

| Host | Role | IP | User |
|------|------|----|------|
| Jump Host | Ansible Controller | — | thor |
| stapp01 | App Server 1 | 172.16.238.10 | tony |
| stapp02 | App Server 2 | 172.16.238.11 | steve |
| stapp03 | App Server 3 | 172.16.238.12 | banner |

---

## 🚀 Solution Walkthrough

### Step 1 — Inspect the Inventory File

```bash
cat /home/thor/ansible/inventory
```

**Initial inventory (missing `ansible_user` — the bug!):**
```ini
stapp01 ansible_host=172.16.238.10 ansible_ssh_pass=Ir0nM@n
stapp02 ansible_host=172.16.238.11 ansible_ssh_pass=Am3ric@
stapp03 ansible_host=172.16.238.12 ansible_ssh_pass=BigGr33n
```

---

### Step 2 — Generate SSH Key Pair on Jump Host

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
```

```
Generating public/private rsa key pair.
Your identification has been saved in /home/thor/.ssh/id_rsa
Your public key has been saved in /home/thor/.ssh/id_rsa.pub
```

---

### Step 3 — Copy Public Key to App Server 3

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub banner@stapp03
```

```
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed
banner@stapp03's password:
Number of key(s) added: 1
```

---

### Step 4 — Verify Password-less SSH

```bash
ssh banner@stapp03
# Logs in directly — no password prompt 
exit
```

---

### Step 5 — First Ansible Ping Attempt (FAILED ❌)

```bash
ansible stapp03 -m ping -i /home/thor/ansible/inventory
```

```
stapp03 | UNREACHABLE! => {
    "msg": "Invalid/incorrect password: Permission denied, please try again.",
    "unreachable": true
}
```

### Root Cause

The inventory was **missing `ansible_user`** for each host. Without it, Ansible defaults to connecting as the current user (`thor`), which doesn't exist on the app servers — so even the correct `ansible_ssh_pass` fails because the username is wrong.

---

### Step 6 — Fix the Inventory (Add `ansible_user`)

```bash
cat > /home/thor/ansible/inventory << 'EOF'
stapp01 ansible_host=172.16.238.10 ansible_user=tony  ansible_ssh_pass=Ir0nM@n
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_pass=Am3ric@
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_pass=BigGr33n
EOF
```

---

### Step 7 — Run Ansible Ping (SUCCESS ✅)

```bash
ansible stapp03 -m ping -i /home/thor/ansible/inventory
```

```json
stapp03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

**`"ping": "pong"` — Lab complete!**

---
