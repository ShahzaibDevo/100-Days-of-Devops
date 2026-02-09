# Jenkins Slave Nodes Configuration

## 📋 Table of Contents
- [Overview](#overview)
- [Lab Environment](#lab-environment)
- [Theory](#theory)
- [Implementation Steps](#implementation-steps)
- [Verification](#verification)

---

## 🎯 Overview

Configure Jenkins master-slave architecture by adding three application servers as SSH build agents for distributed builds and parallel execution.

### Objectives
- Set up SSH authentication between Jenkins and app servers
- Install SSH Build Agents plugin in Jenkins
- Add three app servers as permanent slave nodes
- Configure labels and remote directories
- Verify all nodes are online

---

## 🖥️ Lab Environment

### Server Information

| Server | IP | Hostname | User | Password |
|--------|-----|----------|------|----------|
| Jenkins | 172.16.238.19 | jenkins | jenkins | j@rv!s |
| App Server 1 | 172.16.238.10 | stapp01 | tony | Ir0nM@n |
| App Server 2 | 172.16.238.11 | stapp02 | steve | Am3ric@ |
| App Server 3 | 172.16.238.12 | stapp03 | banner | BigGr33n |

### Required Configuration

| Node Name | Label | Remote Directory | Host |
|-----------|-------|------------------|------|
| App_server_1 | stapp01 | /home/tony/jenkins | stapp01 |
| App_server_2 | stapp02 | /home/steve/jenkins | stapp02 |
| App_server_3 | stapp03 | /home/banner/jenkins | stapp03 |

---

## 📚 Theory

### Jenkins Master-Slave Architecture

```
                    Jenkins Master
                  (Manages & Schedules)
                          |
          +---------------+---------------+
          |               |               |
      Slave 1         Slave 2         Slave 3
     (stapp01)       (stapp02)       (stapp03)
   (Executes Jobs) (Executes Jobs) (Executes Jobs)
```

### Key Concepts

**Master Node:**
- Schedules build jobs
- Monitors slave nodes
- Distributes workloads
- Collects build results
- Serves Jenkins UI

**Slave/Agent Node:**
- Executes build jobs
- Runs on remote servers
- Can have specialized environments
- Scales horizontally

**Labels:**
- Tags to identify nodes
- Target specific nodes for jobs
- Example: `stapp01`, `stapp02`, `stapp03`

**Remote Root Directory:**
- Workspace for Jenkins on slave
- Stores job workspaces and artifacts
- Must be writable by SSH user

**SSH Launch Method:**
- Secure communication via SSH
- Requires SSH key authentication
- Automatically manages agent lifecycle

---

## 🚀 Implementation Steps

### Step 1: Generate SSH Key (PEM Format)

```bash
# Generate SSH key in PEM format
ssh-keygen -t rsa -b 4096 -m PEM -f ~/.ssh/id_rsa -N ""
```

**Verify format:**
```bash
head -1 ~/.ssh/id_rsa
# Must show: -----BEGIN RSA PRIVATE KEY-----
```

> **Important:** Use `-m PEM` flag. Jenkins requires PEM format, not OpenSSH format.

---

### Step 2: Copy SSH Keys to App Servers

```bash
# Copy to App Server 1
ssh-copy-id tony@stapp01
# Enter password: Ir0nM@n

# Copy to App Server 2
ssh-copy-id steve@stapp02
# Enter password: Am3ric@

# Copy to App Server 3
ssh-copy-id banner@stapp03
# Enter password: BigGr33n
```

---

### Step 3: Create Remote Directories

```bash
ssh tony@stapp01 "mkdir -p /home/tony/jenkins"
ssh steve@stapp02 "mkdir -p /home/steve/jenkins"
ssh banner@stapp03 "mkdir -p /home/banner/jenkins"
```

---

### Step 4: Test SSH Connectivity

```bash
ssh tony@stapp01 "hostname"
ssh steve@stapp02 "hostname"
ssh banner@stapp03 "hostname"
```

Should work without asking for password.

---

### Step 5: Install SSH Build Agents Plugin

1. Access Jenkins UI (click Jenkins button, login: admin / Adm!n321)
2. Go to **Manage Jenkins** → **Plugins**
3. Click **Available plugins**
4. Search: `SSH Build Agents`
5. Check the plugin checkbox
6. Click **Install**
7. ✅ Check **"Restart Jenkins when installation is complete and no jobs are running"**
8. Wait for restart (2-3 minutes), then login again

---

### Step 6: Add SSH Credentials

**Get private key:**
```bash
cat ~/.ssh/id_rsa
```
Copy entire output (including BEGIN and END lines)

**Create Credential 1 (tony):**
1. Go to **Manage Jenkins** → **Manage Credentials**
2. Click **(global)** → **Add Credentials**
3. Configure:
   - **Kind:** `SSH Username with private key`
   - **ID:** `tony-ssh`
   - **Username:** `tony`
   - **Private Key:** Select "Enter directly" → Paste private key
4. Click **Create**

**Create Credential 2 (steve):**
- **ID:** `steve-ssh`
- **Username:** `steve`
- **Private Key:** Paste same private key
- Click **Create**

**Create Credential 3 (banner):**
- **ID:** `banner-ssh`
- **Username:** `banner`
- **Private Key:** Paste same private key
- Click **Create**

---

### Step 7: Add Node - App_server_1

1. Go to **Manage Jenkins** → **Manage Nodes and Clouds**
2. Click **New Node**
3. Node name: `App_server_1`
4. Select **Permanent Agent** → Click **Create**
5. Configure:
   - **Remote root directory:** `/home/tony/jenkins`
   - **Labels:** `stapp01`
   - **Usage:** `Use this node as much as possible`
   - **Launch method:** `Launch agents via SSH`
   - **Host:** `stapp01`
   - **Credentials:** `tony-ssh`
   - **Host Key Verification Strategy:** `Non verifying Verification Strategy`
6. Click **Save**

---

### Step 8: Add Node - App_server_2

1. **Manage Jenkins** → **Manage Nodes and Clouds** → **New Node**
2. Node name: `App_server_2`, Type: **Permanent Agent**
3. Configure:
   - **Remote root directory:** `/home/steve/jenkins`
   - **Labels:** `stapp02`
   - **Host:** `stapp02`
   - **Credentials:** `steve-ssh`
   - **Host Key Verification Strategy:** `Non verifying Verification Strategy`
4. Click **Save**

---

### Step 9: Add Node - App_server_3

1. **Manage Jenkins** → **Manage Nodes and Clouds** → **New Node**
2. Node name: `App_server_3`, Type: **Permanent Agent**
3. Configure:
   - **Remote root directory:** `/home/banner/jenkins`
   - **Labels:** `stapp03`
   - **Host:** `stapp03`
   - **Credentials:** `banner-ssh`
   - **Host Key Verification Strategy:** `Non verifying Verification Strategy`
4. Click **Save**

---

### Step 10: Install Java 17 on App Servers

```bash
# Install Java 17 on all servers
echo 'Ir0nM@n' | ssh tony@stapp01 "sudo -S yum install -y java-17-openjdk"
echo 'Am3ric@' | ssh steve@stapp02 "sudo -S yum install -y java-17-openjdk"
echo 'BigGr33n' | ssh banner@stapp03 "sudo -S yum install -y java-17-openjdk"
```

**Verify:**
```bash
ssh tony@stapp01 "java -version"
ssh steve@stapp02 "java -version"
ssh banner@stapp03 "java -version"
```

> **Note:** Jenkins requires Java 17 or higher for agent communication.

---

### Step 11: Relaunch Agents

1. Go to **Manage Jenkins** → **Manage Nodes and Clouds**
2. Click **App_server_1** → **Relaunch agent**
3. Click **App_server_2** → **Relaunch agent**
4. Click **App_server_3** → **Relaunch agent**
5. Wait 30 seconds

---

## ✅ Verification

### Check Node Status

Go to **Manage Jenkins** → **Manage Nodes and Clouds**

**Expected Result:**
```
Name          Architecture   Status      
App_server_1  Linux (amd64)  [online] ✅
App_server_2  Linux (amd64)  [online] ✅
App_server_3  Linux (amd64)  [online] ✅
```

All nodes should show:
- Green checkmark or "In sync" status
- Available disk space
- Low response time

### Check Connection Logs

Click on each node → Click **Log**

**Successful log shows:**
```
[SSH] Opening SSH connection to stapp01:22.
[SSH] Authentication successful.
[SSH] Starting agent process...
Agent successfully connected and online
```

---

## 📝 Command Summary

```bash
# 1. Generate SSH key in PEM format
ssh-keygen -t rsa -b 4096 -m PEM -f ~/.ssh/id_rsa -N ""

# 2. Copy keys to servers
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03

# 3. Create directories
ssh tony@stapp01 "mkdir -p /home/tony/jenkins"
ssh steve@stapp02 "mkdir -p /home/steve/jenkins"
ssh banner@stapp03 "mkdir -p /home/banner/jenkins"

# 4. Test SSH
ssh tony@stapp01 "hostname"
ssh steve@stapp02 "hostname"
ssh banner@stapp03 "hostname"

# 5. Install Java 17
echo 'Ir0nM@n' | ssh tony@stapp01 "sudo -S yum install -y java-17-openjdk"
echo 'Am3ric@' | ssh steve@stapp02 "sudo -S yum install -y java-17-openjdk"
echo 'BigGr33n' | ssh banner@stapp03 "sudo -S yum install -y java-17-openjdk"

# 6. Verify Java
ssh tony@stapp01 "java -version"
ssh steve@stapp02 "java -version"
ssh banner@stapp03 "java -version"
```

---

## 🎯 Final Configuration

| Component | Value |
|-----------|-------|
| Master | jenkins (172.16.238.19) |
| Slave Count | 3 nodes |
| Plugin | SSH Build Agents |
| Authentication | SSH Keys (PEM format) |
| Java Version | Java 17 |

**Nodes:**
- App_server_1 → stapp01 → /home/tony/jenkins
- App_server_2 → stapp02 → /home/steve/jenkins
- App_server_3 → stapp03 → /home/banner/jenkins

---

## ✨ Conclusion

Successfully configured Jenkins master-slave architecture with three slave nodes for distributed builds and parallel job execution.

**Benefits:**
- Distribute workload across multiple servers
- Execute jobs in parallel
- Isolate build environments
- Scale horizontally as needed

---

**Lab Status: Complete ✅**