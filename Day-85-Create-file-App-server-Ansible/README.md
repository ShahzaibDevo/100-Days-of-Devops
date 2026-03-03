
## Day 85: Create Files on App Servers using Ansible

### Lab Overview
- Create an inventory file with all 3 app servers
- Write a playbook to create `/opt/code.txt` with specific permissions and owners per server

---

### Step 1: SSH into the Jump Host

```bash
ssh thor@jump_host
```

---

### Step 2: Create the Playbook Directory

```bash
mkdir -p ~/playbook
cd ~/playbook
```

---

### Step 3: Create the Inventory File

```bash
cat > ~/playbook/inventory << 'EOF'
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_pass=Ir0nM@n
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_pass=Am3ric@
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_pass=BigGr33n

[app_servers:vars]
ansible_become=yes
ansible_become_method=sudo
EOF
```

> 💡 **Note:** Stratos DC standard credentials — verify these match your lab environment.

---

### Step 4: Create the Playbook

```bash
cat > ~/playbook/playbook.yml << 'EOF'
---
- name: Create file on App Server 1 (stapp01)
  hosts: stapp01
  become: yes
  tasks:
    - name: Create /opt/code.txt with correct owner and permissions
      file:
        path: /opt/code.txt
        state: touch
        owner: tony
        group: tony
        mode: "0744"

- name: Create file on App Server 2 (stapp02)
  hosts: stapp02
  become: yes
  tasks:
    - name: Create /opt/code.txt with correct owner and permissions
      file:
        path: /opt/code.txt
        state: touch
        owner: steve
        group: steve
        mode: "0744"

- name: Create file on App Server 3 (stapp03)
  hosts: stapp03
  become: yes
  tasks:
    - name: Create /opt/code.txt with correct owner and permissions
      file:
        path: /opt/code.txt
        state: touch
        owner: banner
        group: banner
        mode: "0744"
EOF
```

---

### Step 5: Verify Your Files

```bash
# Check inventory
cat ~/playbook/inventory

# Check playbook
cat ~/playbook/playbook.yml

# Confirm directory structure
ls -la ~/playbook/
```

---

### Step 6: Test Connectivity (Optional but Recommended)

```bash
cd ~/playbook
ansible -i inventory all -m ping
```

Expected output:
```
stapp01 | SUCCESS => { "ping": "pong" }
stapp02 | SUCCESS => { "ping": "pong" }
stapp03 | SUCCESS => { "ping": "pong" }
```

---

### Step 7: Run the Playbook

```bash
cd ~/playbook
ansible-playbook -i inventory playbook.yml
```

---

### Step 8: Validate the Results

```bash
# Check file on stapp01
ansible -i inventory stapp01 -m shell -a "ls -la /opt/code.txt"

# Check file on stapp02
ansible -i inventory stapp02 -m shell -a "ls -la /opt/code.txt"

# Check file on stapp03
ansible -i inventory stapp03 -m shell -a "ls -la /opt/code.txt"
```
