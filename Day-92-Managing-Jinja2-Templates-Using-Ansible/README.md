# Day 92: Managing Jinja2 Templates Using Ansible 🧩

## Task Overview
Add a **Jinja2 template** for `index.html` into an existing `httpd` Ansible role, deploy it to **App Server 3** with correct ownership and permissions — without hardcoding any server-specific values.

---

## 📋 Requirements

| Requirement | Detail |
|-------------|--------|
| Playbook | Target `stapp03` with `httpd` role |
| Jinja2 Template | `index.html.j2` under `role/httpd/templates/` |
| Template Content | `This file was created using Ansible on {{ inventory_hostname }}` |
| Deploy Path | `/var/www/html/index.html` |
| Permissions | `0655` |
| Owner/Group | `banner` (sudo user of `stapp03`) |

---

## 🛠️ Solution

### Step 1: Update playbook.yml

```bash
cat > ~/ansible/playbook.yml << 'EOF'
---
- name: Run httpd role on App Server 3
  hosts: stapp03
  become: true
  roles:
    - role/httpd
EOF
```

### Step 2: Create the Jinja2 Template

```bash
mkdir -p ~/ansible/role/httpd/templates

cat > ~/ansible/role/httpd/templates/index.html.j2 << 'EOF'
This file was created using Ansible on {{ inventory_hostname }}
EOF
```

### Step 3: Add Template Task to tasks/main.yml

```bash
cat >> ~/ansible/role/httpd/tasks/main.yml << 'EOF'

- name: Deploy index.html from Jinja2 template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: banner
    group: banner
    mode: '0655'
EOF
```

### Step 4: Run the Playbook

```bash
cd ~/ansible
ansible-playbook -i inventory playbook.yml
```

### Step 5: Verify

```bash
ssh banner@stapp03 "cat /var/www/html/index.html && ls -la /var/www/html/index.html"
```

---

## 💡 Key Concepts Learned

**Jinja2 Templates in Ansible:**
The `template` module renders `.j2` files using variables before copying them to the target host. `{{ inventory_hostname }}` resolves dynamically at runtime — no hardcoding needed, same template works across all servers.

**`inventory_hostname` vs `ansible_hostname`:**

| Variable | Value |
|----------|-------|
| `inventory_hostname` | Name as defined in inventory (`stapp03`) |
| `ansible_hostname` | Actual OS-level system hostname |

**Ansible `template` module parameters:**

| Parameter | Purpose |
|-----------|---------|
| `src` | Source `.j2` file (relative to `templates/`) |
| `dest` | Destination path on remote host |
| `owner` | File owner |
| `group` | File group |
| `mode` | File permissions |

**Role `templates/` directory:**
Ansible automatically looks for Jinja2 templates inside the `templates/` folder of a role. Just reference the filename in `src` — no full path needed.

---

## 🔑 Why This Matters

Jinja2 templates turn static config files into **dynamic, environment-aware configurations**. One template serves all servers — Ansible handles variable substitution at runtime. This pattern is used daily for generating Apache/Nginx configs, app config files, environment files, and more in real production infrastructure.

---

