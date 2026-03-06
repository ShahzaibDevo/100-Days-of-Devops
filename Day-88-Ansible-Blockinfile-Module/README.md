#  Day 88 of DevOps Challenge — Ansible `blockinfile` Module

## Task Overview

The **Nautilus DevOps team** needed to deploy and configure a simple `httpd` web server across **all app servers** in **Stratos DC** using Ansible — with a sample web page deployed automatically via the `blockinfile` module.

---

##  Infrastructure

| Server   | Host     | User    |
|----------|----------|---------|
| stapp01  | stapp01  | tony    |
| stapp02  | stapp02  | steve   |
| stapp03  | stapp03  | banner  |

---

##  Requirements

-  Install `httpd` web server on all app servers
- Ensure `httpd` service is **started and enabled**
-  Use `blockinfile` module to add content to `/var/www/html/index.html`
- File owner and group must be `apache`
-  File permissions must be `0744`
- Use **default markers** (no custom/empty markers)

---

##  Inventory File

```ini
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner
```

---

##  Playbook — `playbook.yml`

```yaml
---
- name: Install and configure httpd web server
  hosts: all
  become: yes

  tasks:

    - name: Install httpd package
      yum:
        name: httpd
        state: present

    - name: Start and enable httpd service
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Create index.html file with correct ownership and permissions
      file:
        path: /var/www/html/index.html
        state: touch
        owner: apache
        group: apache
        mode: "0744"

    - name: Add content to index.html using blockinfile
      blockinfile:
        path: /var/www/html/index.html
        block: |
          Welcome to XfusionCorp!
          This is Nautilus sample file, created using Ansible!
          Please do not modify this file manually!

    - name: Ensure correct ownership on index.html
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache
        mode: "0744"
```

---

## ▶ Execution

### Syntax Check
```bash
ansible-playbook -i inventory playbook.yml --syntax-check
# Output: playbook: playbook.yml 
```

### Run the Playbook
```bash
ansible-playbook -i inventory playbook.yml
```


---

##  Idempotency Verified (Run 2)

```
PLAY RECAP *********************************************************************
stapp01 : ok=6  changed=1  unreachable=0  failed=0  skipped=0
stapp02 : ok=6  changed=1  unreachable=0  failed=0  skipped=0
stapp03 : ok=6  changed=1  unreachable=0  failed=0  skipped=0
```

> Only `file: state: touch` shows `changed` on re-run (updates timestamp by design).  
> Everything else is **idempotent** — no unintended changes. 

---

##  Resulting File Content on App Servers

```
# BEGIN ANSIBLE MANAGED BLOCK
Welcome to XfusionCorp!
This is Nautilus sample file, created using Ansible!
Please do not modify this file manually!
# END ANSIBLE MANAGED BLOCK
```

---

##  Key Concepts Learned

### 🔷 `blockinfile` Module
- Inserts or updates a **block of text** inside a file
- Automatically wraps content with **default markers**:
  - `# BEGIN ANSIBLE MANAGED BLOCK`
  - `# END ANSIBLE MANAGED BLOCK`
- Re-running the playbook **updates** the block instead of duplicating it
- Using `marker:` parameter is optional — omitting it uses the safe defaults

### 🔷 Task Ordering Matters
The sequence of tasks is intentional:
1. **Install** httpd → package must exist before we create web content
2. **Start** service → ensures httpd is running
3. **Create** the file → establishes ownership and permissions first
4. **blockinfile** → injects content safely
5. **Re-apply** permissions → guarantees final state is correct

### 🔷 Idempotency
A core Ansible principle — running the same playbook multiple times produces the **same result** without side effects.

---

## 🗺️ Architecture Flow

```
Jump Host (thor)
      │
      │  ansible-playbook -i inventory playbook.yml
      │
      ├──── stapp01 (tony)   → httpd installed + index.html deployed ✅
      ├──── stapp02 (steve)  → httpd installed + index.html deployed ✅
      └──── stapp03 (banner) → httpd installed + index.html deployed ✅
```

---

