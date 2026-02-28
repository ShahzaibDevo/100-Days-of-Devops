H
# Day 82: Create Ansible Inventory for App Server Testing 

## Task Overview
Create an Ansible inventory file on the jump host to enable playbook testing on **App Server 1** in **Stratos DC**.

---

## What is an Ansible Inventory File?
An inventory file is like a **contact list for servers** — it tells Ansible:
- **Where** the server is (IP address)
- **Who** to login as (username)
- **How** to login (password or SSH key)

---

## 🛠️ Solution — Step by Step

### Step 1: Navigate to the playbook directory
```bash
cd /home/thor/playbook/
```

### Step 2: Create the inventory file
```bash
cat > inventory << 'EOF'
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_pass=Ir0nM@n
EOF
```

### Step 3: Verify the file
```bash
cat inventory
```
**Output:**
```
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_pass=Ir0nM@n
```

### Step 4: Test Ansible can read the inventory
```bash
ansible all -i inventory --list-hosts
```
**Output:**
```
  hosts (1):
    stapp01
```

---

## 📋 Inventory File Breakdown

```ini
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_pass=Ir0nM@n
```

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `stapp01` | hostname | Server name per Stratos DC wiki |
| `ansible_host` | `172.16.238.10` | Actual IP of App Server 1 |
| `ansible_user` | `tony` | SSH login username |
| `ansible_ssh_pass` | `Ir0nM@n` | SSH login password |

---

## 🔄 How It Works

```
ansible-playbook -i inventory playbook.yml
        ↓
Ansible reads inventory file
        ↓
Finds stapp01 → IP: 172.16.238.10
        ↓
SSH connects using tony / Ir0nM@n
        ↓
Runs playbook tasks on App Server 1 
```

---

## 💡 Key Concepts Learned

- ✅ **INI format** inventory — simple `key=value` pairs per host
- ✅ **ansible_host** — maps a hostname to an actual IP
- ✅ **ansible_user** — specifies which Linux user to SSH as
- ✅ **ansible_ssh_pass** — enables password-based authentication
- ✅ **Jump Host** — a middle-man machine to reach private network servers

---

