
# Day 84: Copy Data to App Servers using Ansible 

##  Task Overview
The Nautilus DevOps team needs to copy data from the **jump host** to all **application servers** in **Stratos DC** using Ansible.

### Requirements:
- Create an inventory file `/home/thor/ansible/inventory` on jump host
- Create a playbook `/home/thor/ansible/playbook.yml` to copy `/usr/src/dba/index.html` to `/opt/dba` on all app servers

---

##  Environment Details

| Server | Hostname | IP | User |
|--------|----------|----|------|
| App Server 1 | stapp01 | 172.16.238.10 | tony |
| App Server 2 | stapp02 | 172.16.238.11 | steve |
| App Server 3 | stapp03 | 172.16.238.12 | banner |

---

##  Project Structure

```
/home/thor/ansible/
├── ansible.cfg
├── inventory
└── playbook.yml
```

---

## 🔧 Solution

### Step 1: Create the Directory
```bash
mkdir -p /home/thor/ansible
cd /home/thor/ansible
```

### Step 2: Create ansible.cfg
```ini
[defaults]
host_key_checking = False
inventory = inventory
```

### Step 3: Create Inventory File
```ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_password=Ir0nM@n ansible_become=yes ansible_become_pass=Ir0nM@n
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_password=Am3ric@ ansible_become=yes ansible_become_pass=Am3ric@
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_password=BigGr33n ansible_become=yes ansible_become_pass=BigGr33n
```

### Step 4: Create the Playbook
```yaml
---
- name: Copy index.html to all application servers
  hosts: app_servers
  become: yes
  tasks:
    - name: Create destination directory /opt/dba if not exists
      file:
        path: /opt/dba
        state: directory
        mode: '0755'

    - name: Copy index.html to /opt/dba on app servers
      copy:
        src: /usr/src/dba/index.html
        dest: /opt/dba/index.html
        mode: '0644'
```

### Step 5: Run the Playbook
```bash
ansible-playbook -i inventory playbook.yml
```

---

##  Execution Output

```
PLAY [Copy index.html to all application servers] ******************************

TASK [Gathering Facts] *********************************************************
ok: [stapp03]
ok: [stapp01]
ok: [stapp02]

TASK [Create destination directory /opt/dba if not exists] *********************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Copy index.html to /opt/dba on app servers] ******************************
changed: [stapp03]
changed: [stapp01]
changed: [stapp02]

PLAY RECAP *********************************************************************
stapp01   : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02   : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03   : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

---

## 🔍 Validation

```bash
ansible app_servers -i inventory -m shell -a "ls -la /opt/dba/index.html" --become
```

```
stapp01 | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 35 Mar  2 10:08 /opt/dba/index.html

stapp02 | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 35 Mar  2 10:08 /opt/dba/index.html

stapp03 | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 35 Mar  2 10:08 /opt/dba/index.html
```

---

## 📊Results Summary

| Check | Result |
|-------|--------|
| Inventory created | ✅ |
| Playbook executed | ✅ |
| Facts gathered (all 3 servers) | ✅ |
| `/opt/dba` directory exists | ✅ |
| `index.html` copied to all servers | ✅ |
| Zero failures / unreachable | ✅ |
| File size matches source (35 bytes) | ✅ |

---

##  Key Concepts Learned

### Ansible `copy` Module
- Copies files **from the control node** (jump host) to managed nodes
- Key parameters:
  - `src` — source path on the control node
  - `dest` — destination path on the managed node
  - `mode` — file permissions

### Ansible `file` Module
- Used to create/manage directories
- `state: directory` ensures the directory exists before copying

### `become: yes`
- Enables privilege escalation (sudo)
- Required for writing to system directories like `/opt/`

### `host_key_checking = False`
- Disables SSH fingerprint verification
- Useful in lab environments to prevent connection prompts

---

