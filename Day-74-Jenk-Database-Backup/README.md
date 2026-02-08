# Day 74: Jenkins Database Backup Job

## Lab Scenario

There is a requirement to create a Jenkins job to automate the database backup. The backup job should connect to the Database server in Stratos Datacenter, take a dump of the `kodekloud_db01` database, and copy it to the Backup server at regular intervals.

## Lab Requirements

1. Create a Jenkins job named `database-backup`
2. Configure it to take a database dump of the `kodekloud_db01` database present on the Database server
3. Database credentials:
   - Database User: `kodekloud_roy`
   - Password: `asdfgdsd`
4. The dump should be named in `db_$(date +%F).sql` format (e.g., `db_2026-02-01.sql`)
5. Copy the dump to the Backup Server under location `/home/clint/db_backups`
6. Schedule this job to run periodically at `*/10 * * * *` (every 10 minutes)



### Key Servers for This Lab

- **Database Server**: stdb01 (172.16.239.10) - SSH User: `peter`, Password: `Sp!dy`
- **Backup Server**: stbkp01 (172.16.238.16) - SSH User: `clint`, Password: `H@wk3y3`
- **Jenkins Server**: jenkins (172.16.238.19) - SSH User: `jenkins`, Password: `j@rv!s`

### Database Information

- **Database Name**: `kodekloud_db01`
- **Database User**: `kodekloud_roy`
- **Database Password**: `asdfgdsd`

## Solution

### Architecture Overview

```
Jenkins Server (172.16.238.19)
    |
    |-- SSH --> Database Server (stdb01 - 172.16.239.10)
    |              User: peter
    |              Runs: mysqldump (as kodekloud_roy)
    |              Creates: /tmp/db_YYYY-MM-DD.sql
    |
    └-- SCP --> Backup Server (stbkp01 - 172.16.238.16)
                   User: clint
                   Destination: /home/clint/db_backups/
```

### Step 1: Access Jenkins

1. Click on the **Jenkins** button on the top bar to access the Jenkins UI
2. Login using credentials:
   - Username: `admin`
   - Password: `Adm!n321`

### Step 2: Setup SSH Key Authentication

SSH key authentication is required between Jenkins server and both Database/Backup servers.

#### 2.1 SSH to Jenkins Server

```bash
ssh jenkins@jenkins
# Password: j@rv!s
```

#### 2.2 Generate SSH Key Pair

```bash
ssh-keygen -t rsa -b 2048 -N "" -f ~/.ssh/id_rsa
```

Output:
```
Generating public/private rsa key pair.
Created directory '/var/lib/jenkins/.ssh'.
Your identification has been saved in /var/lib/jenkins/.ssh/id_rsa
Your public key has been saved in /var/lib/jenkins/.ssh/id_rsa.pub
```

#### 2.3 View the Public Key

```bash
cat ~/.ssh/id_rsa.pub
```

Copy the entire output (starts with `ssh-rsa`). Example:
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCzL1sYXlPVZNG9c9Vkwp8+b3nhPZQ07n9+YgD+S5B75oXeKSlc0eC2RhN4hj9xe+KA6q7eKCGeY7yHyGmJNzYhSsOwFBkL0qN74znSKciziGzl7PzVFQ8kvEraUcB3/CzZNeVSYWRW+/Oj4jvY2BoiP5ayoIikQSGTIfjrdYzjEovva5cLpDlmF/n37/kN2R2BmappaTSPx3YHON9xUmQwV25PIAQ/Fe7+9rmajxQLccGP3kzZLtK+L28fswOvJiVek7IBG3SaC2vLRL4yQ/hDRdyzSxr1PlVKs0Kwf+zGWcwjhvmE+iIWS503iHTPSirbNnk0Xhh76E/8nRDaHNGI+hXM3col+VV9SKHDF/0ONQUc7M9IWKu1BRxyM20TqUl6l7cU9ZhmhcZsujg+fuW/U6uKUODF6FIiBJXWZV300jAmrWvLsoTFwVNyFcy/5UtytcCVyodu221DLtXIRhm+jDROxuyOwoLmHSudd5XIKZhb9Qq7Fg+iMShr4lFv7pU= jenkins@jenkins.stratos.xfusioncorp.com
```

#### 2.4 Add Public Key to Database Server

From Jenkins server, SSH to the database server:

```bash
ssh peter@stdb01
# Password: Sp!dy
```

Add the Jenkins public key to authorized_keys:

```bash
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCzL1sYXlPVZNG9c9Vkwp8+b3nhPZQ07n9+YgD+S5B75oXeKSlc0eC2RhN4hj9xe+KA6q7eKCGeY7yHyGmJNzYhSsOwFBkL0qN74znSKciziGzl7PzVFQ8kvEraUcB3/CzZNeVSYWRW+/Oj4jvY2BoiP5ayoIikQSGTIfjrdYzjEovva5cLpDlmF/n37/kN2R2BmappaTSPx3YHON9xUmQwV25PIAQ/Fe7+9rmajxQLccGP3kzZLtK+L28fswOvJiVek7IBG3SaC2vLRL4yQ/hDRdyzSxr1PlVKs0Kwf+zGWcwjhvmE+iIWS503iHTPSirbNnk0Xhh76E/8nRDaHNGI+hXM3col+VV9SKHDF/0ONQUc7M9IWKu1BRxyM20TqUl6l7cU9ZhmhcZsujg+fuW/U6uKUODF6FIiBJXWZV300jAmrWvLsoTFwVNyFcy/5UtytcCVyodu221DLtXIRhm+jDROxuyOwoLmHSudd5XIKZhb9Qq7Fg+iMShr4lFv7pU= jenkins@jenkins.stratos.xfusioncorp.com" > ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys
exit
```

#### 2.5 Add Public Key to Backup Server

From Jenkins server, SSH to the backup server:

```bash
ssh clint@stbkp01
# Password: H@wk3y3
```

Add the Jenkins public key to authorized_keys:

```bash
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCzL1sYXlPVZNG9c9Vkwp8+b3nhPZQ07n9+YgD+S5B75oXeKSlc0eC2RhN4hj9xe+KA6q7eKCGeY7yHyGmJNzYhSsOwFBkL0qN74znSKciziGzl7PzVFQ8kvEraUcB3/CzZNeVSYWRW+/Oj4jvY2BoiP5ayoIikQSGTIfjrdYzjEovva5cLpDlmF/n37/kN2R2BmappaTSPx3YHON9xUmQwV25PIAQ/Fe7+9rmajxQLccGP3kzZLtK+L28fswOvJiVek7IBG3SaC2vLRL4yQ/hDRdyzSxr1PlVKs0Kwf+zGWcwjhvmE+iIWS503iHTPSirbNnk0Xhh76E/8nRDaHNGI+hXM3col+VV9SKHDF/0ONQUc7M9IWKu1BRxyM20TqUl6l7cU9ZhmhcZsujg+fuW/U6uKUODF6FIiBJXWZV300jAmrWvLsoTFwVNyFcy/5UtytcCVyodu221DLtXIRhm+jDROxuyOwoLmHSudd5XIKZhb9Qq7Fg+iMShr4lFv7pU= jenkins@jenkins.stratos.xfusioncorp.com" > ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys
exit
```

#### 2.6 Test SSH Connections

Back on Jenkins server, test passwordless SSH:

```bash
# Test database server
ssh -o StrictHostKeyChecking=no peter@stdb01 "hostname"
# Output: stdb01.stratos.xfusioncorp.com (no password prompt!)

# Test backup server
ssh -o StrictHostKeyChecking=no clint@stbkp01 "hostname"
# Output: stbkp01.stratos.xfusioncorp.com (no password prompt!)

exit
```

### Step 3: Create Jenkins Job

#### 3.1 Create New Job

1. From Jenkins Dashboard, click **New Item**
2. Enter job name: `database-backup`
3. Select **Freestyle project**
4. Click **OK**

#### 3.2 Configure General Settings

In the job configuration page:
- **Description**: `Automated database backup job for kodekloud_db01`
- Leave other general settings as default

#### 3.3 Configure Build Triggers

1. Scroll to **Build Triggers** section
2. Check ☑ **Build periodically**
3. In the **Schedule** field, enter:
   ```
   */10 * * * *
   ```
   This will run the job every 10 minutes.

#### 3.4 Configure Build Steps

1. In **Build** section, click **Add build step**
2. Select **Execute shell**
3. Enter the following script:

```bash
#!/bin/bash
set -e

# ============================================
# CONFIGURATION
# ============================================
DB_SERVER="stdb01"
DB_SSH_USER="peter"

BACKUP_SERVER="stbkp01"
BACKUP_USER="clint"

DB_USER="kodekloud_roy"
DB_PASS="asdfgdsd"
DB_NAME="kodekloud_db01"

DUMP_FILE="db_$(date +%F).sql"

# SSH options to bypass host key verification
SSH_OPTIONS="-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"

# ============================================
# BACKUP PROCESS
# ============================================

echo "=========================================="
echo "Starting backup for database: ${DB_NAME}"
echo "Date: $(date)"
echo "Database Server: ${DB_SERVER}"
echo "Backup Server: ${BACKUP_SERVER}"
echo "=========================================="

# Step 1: Create database dump
echo "Step 1: Creating database dump on ${DB_SERVER}..."
ssh ${SSH_OPTIONS} ${DB_SSH_USER}@${DB_SERVER} "mysqldump -u ${DB_USER} -p${DB_PASS} ${DB_NAME}" > /tmp/${DUMP_FILE}

# Step 2: Verify dump file was created
if [ ! -f /tmp/${DUMP_FILE} ]; then
    echo "ERROR: Dump file not created!"
    exit 1
fi

FILE_SIZE=$(du -h /tmp/${DUMP_FILE} | cut -f1)
echo "✓ Dump created successfully: ${DUMP_FILE} (Size: ${FILE_SIZE})"

# Step 3: Copy dump to backup server
echo "Step 2: Copying dump to backup server ${BACKUP_SERVER}..."
scp ${SSH_OPTIONS} /tmp/${DUMP_FILE} ${BACKUP_USER}@${BACKUP_SERVER}:/home/clint/db_backups/

# Step 4: Verify copy succeeded
if [ $? -eq 0 ]; then
    echo "✓ Backup successfully copied to ${BACKUP_SERVER}:/home/clint/db_backups/"
else
    echo "ERROR: Failed to copy backup to ${BACKUP_SERVER}"
    exit 1
fi

# Step 5: Clean up temporary file
echo "Step 3: Cleaning up temporary files..."
rm -f /tmp/${DUMP_FILE}
echo "✓ Temporary files cleaned up"

# Step 6: Success message
echo "=========================================="
echo "✓✓✓ BACKUP COMPLETED SUCCESSFULLY! ✓✓✓"
echo "Backup file: ${DUMP_FILE}"
echo "Location: ${BACKUP_SERVER}:/home/clint/db_backups/"
echo "=========================================="
```

#### 3.5 Save the Job

Click **Save** at the bottom of the page.

### Step 4: Test the Job

#### 4.1 Manual Test Run

1. From the job page, click **Build Now**
2. Watch the **Build History** for the build number
3. Click on the build number (e.g., #1)
4. Click **Console Output** to view the execution logs
