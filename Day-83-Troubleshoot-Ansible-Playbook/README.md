


# Day 83: Troubleshoot and Create Ansible Playbook 

##  Lab Overview
Troubleshoot an incomplete Ansible playbook on the jump host and create a 
playbook to create an empty file on a target App Server in Stratos DC.

---

## Task Requirements
1. Update inventory file `/home/thor/ansible/inventory` for the target App Server
2. Create playbook `/home/thor/ansible/playbook.yml` to create `/tmp/file.txt`
3. Playbook must run using: `ansible-playbook -i inventory playbook.yml`

---

## 🖥️ Stratos DC Credentials (Save This!)

| Server  | IP             | User   | Password    | Description     |
|---------|----------------|--------|-------------|-----------------|
| stapp01 | 172.16.238.10  | tony   | `Ir0nM@n`   | Nautilus App 1  |
| stapp02 | 172.16.238.11  | steve  | `Am3ric@`   | Nautilus App 2  |
| stapp03 | 172.16.238.12  | banner | `BigGr33n`  | Nautilus App 3  |

---

## Solution

### Step 1: Update Inventory File
```bash
cat > /home/thor/ansible/inventory << 'EOF'
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_pass=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
EOF
```

### Step 2: Create the Playbook
```bash
cat > /home/thor/ansible/playbook.yml << 'EOF'
---
- name: Create empty file on App Server 2
  hosts: stapp02
  become: yes
  tasks:
    - name: Create /tmp/file.txt
      file:
        path: /tmp/file.txt
        state: touch
EOF
```

### Step 3: Syntax Check
```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml --syntax-check
```

### Step 4: Run the Playbook
```bash
ansible-playbook -i inventory playbook.yml
```

---

## Expected Output
```
PLAY [Create empty file on App Server 2] ***************************************

TASK [Gathering Facts] *********************************************************
ok: [stapp02]

TASK [Create /tmp/file.txt] ****************************************************
changed: [stapp02]

PLAY RECAP *********************************************************************
stapp02 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    
```

---

##  Key Concepts Learned
- Ansible inventory file structure and SSH authentication parameters
- Using `ansible_ssh_pass` for password-based authentication
- Using `ansible_ssh_common_args` to bypass host key checking
- Ansible `file` module with `state: touch` to create empty files
- Using `become: yes` for privilege escalation on remote hosts
- Always run `--syntax-check` before executing playbooks

---
