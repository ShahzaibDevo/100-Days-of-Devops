# Day 91: Ansible Lineinfile Module

## 📋 Lab Task

The **Nautilus DevOps team** wants to install and set up a simple `httpd` web server on all app servers in **Stratos DC** and deploy a sample web page using Ansible.

### Requirements
1. Install `httpd` on all app servers and ensure the service is **started and enabled**
2. Create `/var/www/html/index.html` with content:
   ```
   This is a Nautilus sample file, created using Ansible!
   ```
3. Use the **`lineinfile`** module to add the following line **at the top** of the file:
   ```
   Welcome to xFusionCorp Industries!
   ```
4. Set file **owner and group** to `apache` on all app servers
5. Set file **permissions** to `0744` on all app servers

---

## 🗂️ Inventory

```ini
# /home/thor/ansible/inventory
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner
```

---

## 📝 Playbook — `playbook.yml`

```yaml
---
- name: Setup httpd web server with sample page
  hosts: all
  become: yes

  tasks:

    - name: Install httpd web server
      package:
        name: httpd
        state: present

    - name: Start and enable httpd service
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Create index.html with initial content
      copy:
        dest: /var/www/html/index.html
        content: "This is a Nautilus sample file, created using Ansible!"
        owner: apache
        group: apache
        mode: "0744"

    - name: Add welcome line at the top using lineinfile
      lineinfile:
        path: /var/www/html/index.html
        line: "Welcome to xFusionCorp Industries!"
        insertbefore: BOF

    - name: Ensure correct owner and group on index.html
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache
        mode: "0744"
```

---

## ▶️ Execution

**Syntax check:**
```bash
ansible-playbook -i inventory playbook.yml --syntax-check
```

**Run the playbook:**
```bash
ansible-playbook -i inventory playbook.yml
```


## ✅ Verification

SSH into an app server and verify:

```bash
ssh tony@stapp01
cat /var/www/html/index.html
ls -la /var/www/html/index.html
```
ns | `0744` (`-rwxr--r--`) | ✅ |

---

## 🔑 Key Concepts

### `lineinfile` module
The `lineinfile` module manages lines in a text file. Key parameters used here:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `path` | `/var/www/html/index.html` | Target file |
| `line` | `"Welcome to xFusionCorp Industries!"` | Line to insert |
| `insertbefore` | `BOF` | Insert at **B**eginning **O**f **F**ile |

> **`insertbefore: BOF`** ensures the new line is placed at the very top of the file, above all existing content.

### Modules Used

| Module | Purpose |
|--------|---------|
| `package` | Install `httpd` (distro-agnostic) |
| `service` | Start and enable `httpd` |
| `copy` | Create `index.html` with initial content |
| `lineinfile` | Add a line at the top of the file |
| `file` | Enforce final ownership and permissions |

---
