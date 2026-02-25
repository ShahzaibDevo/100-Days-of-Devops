# Day 79: Jenkins Deployment Job 🚀

## Overview

This task involves setting up a **CI/CD pipeline** using Jenkins to automatically deploy web application changes from a Git repository to app servers whenever a developer pushes to the `master` branch.

### Infrastructure
| Component | Host | User | Password |
|-----------|------|------|----------|
| Jenkins Server | jenkins.stratos.xfusioncorp.com | admin | Adm!n321 |
| Git Server (Gitea) | git.stratos.xfusioncorp.com | sarah | Sarah_pass123 |
| Storage Server | ststor01 | sarah | Sarah_pass123 |
| Storage Server | ststor01 | natasha | Bl@kW |
| App Server 1 | stapp01 | tony | Ir0nM@n |
| App Server 2 | stapp02 | steve | Am3ric@ |
| App Server 3 | stapp03 | banner | BigGr33n |
| Jump Host | thor | thor | mjolnir123 |

---

## Step 1: Install httpd on All App Servers

SSH into each app server and install + configure Apache httpd to serve on port **8080**.

### App Server 1 (stapp01)
```bash
ssh tony@stapp01
# Password: Ir0nM@n

sudo yum install -y httpd
sudo sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf
sudo systemctl enable httpd
sudo systemctl start httpd
exit
```

### App Server 2 (stapp02)
```bash
ssh steve@stapp02
# Password: Am3ric@

sudo yum install -y httpd
sudo sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf
sudo systemctl enable httpd
sudo systemctl start httpd
exit
```

### App Server 3 (stapp03)
```bash
ssh banner@stapp03
# Password: BigGr33n

sudo yum install -y httpd
sudo sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf
sudo systemctl enable httpd
sudo systemctl start httpd
exit
```

---

## Step 2: Fix Permissions on Storage Server

The `/var/www/html` directory is shared via NFS to all app servers. Sarah needs ownership of this directory so Jenkins can deploy files there.

### Access Storage Server via Natasha (Admin User)

From the **Thor jumphost** terminal:
```bash
ssh natasha@ststor01
# Password: Bl@kW
```

### Grant sarah sudo access and fix ownership
```bash
sudo chown -R sarah:sarah /var/www/html
sudo chmod 755 /var/www/html
echo "sarah ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/sarah
sudo chmod 440 /etc/sudoers.d/sarah
```

### Verify
```bash
ls -la /var/www/
# Expected: drwxr-xr-x sarah sarah html
```

---

## Step 3: Install Jenkins Plugin

1. Go to **Jenkins UI** → `Manage Jenkins` → `Manage Plugins` → `Available`
2. Search for **"Publish Over SSH"**
3. Install it
4. Check **"Restart Jenkins when installation is complete and no jobs are running"**
5. Wait for Jenkins to restart and refresh the page

---

## Step 4: Configure Publish Over SSH in Jenkins

1. Go to **Manage Jenkins** → **Configure System**
2. Scroll down to **"Publish over SSH"** section
3. Under **SSH Servers**, click **Add**
4. Fill in:

| Field | Value |
|-------|-------|
| Name | `Storage Server` |
| Hostname | `ststor01` |
| Username | `sarah` |
| Remote Directory | `/home/sarah` |
| Authentication | Password |
| Password | `Sarah_pass123` |

5. Click **Test Configuration** — should say "Success"
6. Click **Save**

---

## Step 5: Create Jenkins Job

1. From Jenkins dashboard, click **New Item**
2. Enter name: `nautilus-app-deployment`
3. Select **Freestyle project**
4. Click **OK**

### Source Code Management
- Select **Git**
- Repository URL: `http://git.stratos.xfusioncorp.com/sarah/web.git`
- Branch Specifier: `*/master`
- Credentials: None (public repo) or add sarah's credentials

### Build Triggers
- Check **"Poll SCM"**
- Schedule: `* * * * *` (polls every minute)

> **Alternatively**, set up a Gitea Webhook (see Step 6) for instant triggering.

### Build Steps → Send files or execute commands over SSH

Click **Add build step** → **Send files or execute commands over SSH**

| Field | Value |
|-------|-------|
| SSH Server | `Storage Server` |
| Source files | `**/*` |
| Remove prefix | *(leave blank)* |
| Remote directory | *(leave blank)* |
| Exec command | `cd /var/www/html && git pull origin master` |

> **Why `git pull`?** The `/var/www/html` directory on the storage server is already a cloned Git repository. Instead of copying files, we simply pull the latest changes directly — this is cleaner and ensures all repo contents are deployed.

6. Click **Save**

---

## Step 6: Set Up Gitea Webhook (Auto-trigger on Push)

1. Go to **Gitea** → Login as `sarah`
2. Navigate to the `web` repository
3. Go to **Settings** → **Webhooks** → **Add Webhook** → **Gitea**
4. Fill in:

| Field | Value |
|-------|-------|
| Target URL | `http://admin:Adm!n321@jenkins.stratos.xfusioncorp.com/gitea-webhook/post` |
| Content Type | `application/json` |
| Trigger On | Push Events |
| Active | ✅ Checked |

5. Click **Add Webhook**
6. Click **Test Delivery** to verify it triggers Jenkins

---

## Step 7: Update index.html and Push

SSH into the storage server as sarah:

```bash
ssh sarah@ststor01
# Password: Sarah_pass123

cd ~/web
echo "Welcome to the xFusionCorp Industries" > index.html

git config user.email "sarah@stratos.xfusioncorp.com"
git config user.name "sarah"

git add -A
git commit -m "Update index.html"
git push origin master
```

This push will **automatically trigger** the Jenkins job via the webhook.

---

## Step 8: Verify Jenkins Build

1. Go to **Jenkins** → `nautilus-app-deployment`
2. Watch the build trigger automatically (or click **Build Now**)
3. Click on the build number → **Console Output**

### Expected Successful Output
```
Started by an SCM change
...
SSH: Connecting from host [jenkins.stratos.xfusioncorp.com]
SSH: Connecting with configuration [Storage Server] ...
SSH: EXEC: completed after 201 ms
SSH: Disconnecting configuration [Storage Server] ...
SSH: Transferred 1 file(s)
Build step 'Send files or execute commands over SSH' changed build result to SUCCESS
Finished: SUCCESS
```

---

## Step 9: Verify Deployment

### Check on Storage Server
```bash
ssh sarah@ststor01

cat /var/www/html/index.html
# Expected output: Welcome to the xFusionCorp Industries

ls -la /var/www/html/
# Should show index.html owned by sarah
```

### Check the App URL
Click the **App** button in the KodeKloud lab interface. You should see:

```
Welcome to the xFusionCorp Industries
```

---

## Troubleshooting

### Error: "Could not create or change to directory. Directory [var]"
**Cause:** Remote Directory in SSH build step set to `/var/www/html` — treated as relative path.  
**Fix:** Set Remote Directory in **global SSH config** to `/home/sarah`, leave it blank in the job.

### Error: "Exec exit status not zero. Status [1]"
**Cause:** Permission denied — sarah can't write to `/var/www/html`.  
**Fix:** From natasha on ststor01:
```bash
sudo chown -R sarah:sarah /var/www/html
echo "sarah ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/sarah
```

### Error: "fatal: detected dubious ownership in repository"
**Cause:** Git safety check when running as different user.  
**Fix:** Run as sarah (not natasha/root), or add:
```bash
git config --global --add safe.directory /var/www/html
```

### Root SSH Blocked
Root password login is disabled on ststor01. Always use `natasha` user with sudo.

---

## Architecture Diagram

```
Developer (sarah)
      |
      | git push
      ▼
  Gitea Repo
  (web/master)
      |
      | Webhook POST
      ▼
  Jenkins Job
  (nautilus-app-deployment)
      |
      | Publish Over SSH
      | (git pull)
      ▼
  Storage Server (ststor01)
  /var/www/html/  ←── NFS Share
      |
      | NFS Mount
      ▼
  ┌─────────────────────────────┐
  │  stapp01  stapp02  stapp03  │
  │  httpd    httpd    httpd    │
  │  :8080    :8080    :8080    │
  └─────────────────────────────┘
      |
      ▼
  Load Balancer → App URL
```

---

## Key Takeaways

- `/var/www/html` on the storage server is an **NFS share** mounted on all app servers — deploy once, serve everywhere
- Jenkins uses **Publish Over SSH** plugin to connect to the storage server and trigger `git pull`
- The Gitea **webhook** enables true CI/CD — every push automatically triggers deployment
- Always ensure the **deploy user (sarah) owns** the deployment directory
- Use `natasha` (the admin user) to fix permissions on ststor01 since root SSH login is disabled

---
