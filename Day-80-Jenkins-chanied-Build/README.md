
# Day 80: Jenkins Chained Builds 

## Prerequisites
- Jenkins UI: login `admin` / `Adm!n321`
- Gitea UI (port 8090): login `sarah` / `Sarah_pass123`
- Storage Server `ststor01`: user `natasha` / `Bl@kW`
- App Servers: `stapp01` (tony/`Ir0nM@n`), `stapp02` (steve/`Am3ric@`), `stapp03` (banner/`BigGr33n`)

---

## Step 1: Install `sshpass` Plugin on Jenkins

1. Go to **Manage Jenkins** → **Manage Plugins** → **Available**
2. Search for **SSH Agent** → Install
3. Click **Restart Jenkins when installation is complete**

---

## Step 2: Verify `sshpass` on Jenkins Server

SSH into Jenkins server and run:
```bash
which sshpass
# If not found:
sudo yum install sshpass -y
```

---

## Step 3: Create Job 1 — `nautilus-app-deployment`

1. Jenkins Dashboard → **New Item**
2. Name: `nautilus-app-deployment`
3. Select **Freestyle project** → **OK**

### Build Section → Add **Execute shell**:
```bash
#!/bin/bash
sshpass -p 'Bl@kW' ssh -o StrictHostKeyChecking=no \
  -o PreferredAuthentications=password natasha@ststor01 \
  "echo 'Bl@kW' | sudo -S bash -c 'cd /var/www/html && git fetch origin && git reset --hard origin/master'"
```

### Post-build Actions:
1. Click **Add post-build action** → **Build other projects**
2. Projects to build: `manage-services`
3. Select:  **Trigger only if build is stable**

4. Click **Save**

---

## Step 4: Create Job 2 — `manage-services`

1. Jenkins Dashboard → **New Item**
2. Name: `manage-services`
3. Select **Freestyle project** → **OK**

### Build Triggers:
1. Check  **Build after other projects are built**
2. Projects to watch: `nautilus-app-deployment`
3. Select:  **Trigger only if build is stable**

### Build Section → Add **Execute shell**:
```bash
#!/bin/bash
sshpass -p 'Ir0nM@n' ssh -o StrictHostKeyChecking=no \
  -o PreferredAuthentications=password tony@stapp01 \
  "echo 'Ir0nM@n' | sudo -S systemctl restart httpd"

sshpass -p 'Am3ric@' ssh -o StrictHostKeyChecking=no \
  -o PreferredAuthentications=password steve@stapp02 \
  "echo 'Am3ric@' | sudo -S systemctl restart httpd"

sshpass -p 'BigGr33n' ssh -o StrictHostKeyChecking=no \
  -o PreferredAuthentications=password banner@stapp03 \
  "echo 'BigGr33n' | sudo -S systemctl restart httpd"

echo "httpd restarted on all app servers successfully"
```

4. Click **Save**

---

## Step 5: Run and Verify

1. Go to `nautilus-app-deployment` → Click **Build Now**
2. Check **Console Output** — you should see:
```
HEAD is now at xxxxxxx Updated index.html file
Triggering a new build of manage-services
Finished: SUCCESS
```
3. Go to `manage-services` → Check **Console Output**:
```
httpd restarted on all app servers successfully
Finished: SUCCESS
```

---

## Step 6: Verify App is Live

1. Click the **App** button on the top bar
2. Confirm latest content loads on `https://<LBR-URL>`
3. Should **not** require a subdirectory like `/web`

---

## Common Errors & Fixes

| Error | Fix |
|-------|-----|
| `Permission denied (publickey)` | Add `-o PreferredAuthentications=password` and use `sshpass` |
| `sudo: a terminal is required` | Use `echo 'password' \| sudo -S` |
| `pipe \| running locally not remotely` | Wrap entire remote command in double quotes `"..."` |
| `natasha sudo error` | Use `sudo -S bash -c '...'` to run all commands in one sudo call |

---

## Architecture Flow

```
[Jenkins] 
    │
    ▼
nautilus-app-deployment (Job 1)
    │  - SSH to ststor01
    │  - git pull /var/www/html
    │  - Shared volume auto reflects on stapp01/02/03
    │
    ▼ (only if STABLE)
manage-services (Job 2)
    │  - SSH to stapp01 → restart httpd
    │  - SSH to stapp02 → restart httpd
    │  - SSH to stapp03 → restart httpd
    ▼
App Live on LB URL 
```

---
