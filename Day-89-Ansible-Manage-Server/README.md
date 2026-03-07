# Day 89: Ansible Manage Services

## Task
Install `httpd` on all app servers, then start and enable the service using Ansible.

---

## Step 1 — Check the Inventory

```bash
cat /home/thor/ansible/inventory
```

---

## Step 2 — Configure ansible.cfg

```bash
vi /home/thor/ansible/ansible.cfg
```

```ini
[defaults]
host_key_checking = False
```

---

## Step 3 — Create the Playbook

```bash
vi /home/thor/ansible/playbook.yml
```

```yaml
---
- name: Install and manage httpd service on all app servers
  hosts: all
  become: yes
  tasks:

    - name: Install httpd package
      yum:
        name: httpd
        state: present

    - name: Start httpd service
      service:
        name: httpd
        state: started

    - name: Enable httpd service on boot
      service:
        name: httpd
        enabled: yes
```

---

## Step 4 — Syntax Check

```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml --syntax-check
```

---

## Step 5 — Run the Playbook

```bash
ansible-playbook -i inventory playbook.yml
```

---

## Step 6 — Verify on App Server

```bash
ssh tony@stapp01
sudo systemctl status httpd
```

---

