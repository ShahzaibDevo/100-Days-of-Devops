# Day 87: Ansible Install Package 

## Task Overview

The Nautilus Application Development team needed specific packages installed across all app servers in the **Stratos Datacenter**. Since Ansible is already in use for automation, the task was to write a playbook that installs `wget` on all app servers — without manual SSH-ing into each one.

---

##  Objectives

1. Create an inventory file at `/home/thor/playbook/inventory` on the jump host with all app servers listed.
2. Create an Ansible playbook at `/home/thor/playbook/playbook.yml` to install `wget` using the Ansible `yum` module.
3. Ensure the playbook runs successfully with just:
   ```bash
   ansible-playbook -i inventory playbook.yml
   ```

---

## 🖥️ Environment

| Server   | IP Address      | User    | Password    |
|----------|-----------------|---------|-------------|
| stapp01  | 172.16.238.10   | tony    | Ir0nM@n     |
| stapp02  | 172.16.238.11   | steve   | Am3ric@     |
| stapp03  | 172.16.238.12   | banner  | BigGr33n    |

> Jump Host User: `thor`

---

## Step 1: Create the Inventory File

```bash
mkdir -p /home/thor/playbook
cd /home/thor/playbook
```

```bash
cat > /home/thor/playbook/inventory << 'EOF'
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_become=yes ansible_become_pass=Ir0nM@n ansible_python_interpreter=/usr/bin/python3
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_pass=Am3ric@ ansible_become=yes ansible_become_pass=Am3ric@ ansible_python_interpreter=/usr/bin/python3
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_pass=BigGr33n ansible_become=yes ansible_become_pass=BigGr33n ansible_python_interpreter=/usr/bin/python3
EOF
```

---

##  Step 2: Create the Ansible Playbook

>  **Important:** The `yum` module caused **rc=137 (OOM Kill)** errors on stapp01 and stapp03 due to low memory on those hosts. The fix was to use the `raw` module, which executes commands directly over SSH with **zero Python overhead**.

```bash
cat > /home/thor/playbook/playbook.yml << 'EOF'
---
- name: Install wget on all app servers
  hosts: all
  become: yes
  gather_facts: false
  tasks:
    - name: Install wget package using raw
      raw: yum install -y wget
EOF
```

---

##  Step 3: Verify Connectivity

```bash
ansible -i inventory all -m ping
```

**Output:**
```
stapp01 | SUCCESS => { "ping": "pong" }
stapp02 | SUCCESS => { "ping": "pong" }
stapp03 | SUCCESS => { "ping": "pong" }
```

---

## 🚀 Step 4: Run the Playbook

```bash
ansible-playbook -i inventory playbook.yml
```
