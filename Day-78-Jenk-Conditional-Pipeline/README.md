** Day 78: Jenkins Conditional Pipeline** lab:

---

## Step 1: Access the UIs
- Jenkins: login as `admin` / `Adm!n321`
- Gitea: login as `sarah` / `Sarah_pass123`

---

## Step 2: Install SSH Build Agents Plugin
1. **Manage Jenkins** → **Manage Plugins** → **Available**
2. Search **SSH Build Agents** → Install
3. Click **Restart Jenkins when installation is complete**

---

## Step 3: Add Storage Server Slave Node
1. **Manage Jenkins** → **Manage Nodes** → **New Node**
2. Fill in:
```
Node name:              Storage Server
Type:                   Permanent Agent
Labels:                 ststor01
Remote root directory:  /var/www/html
Launch method:          Launch agents via SSH
Host:                   172.16.238.15
```
3. Add Credentials:
```
Kind:        Username with password
Username:    natasha
Password:    Bl@kW
ID:          ststor01
```
4. **Host Key Verification Strategy**: `Non verifying Verification Strategy`
5. Click **Save**

---

## Step 4: Fix Permissions on Storage Server
SSH into storage server and fix ownership:
```bash
ssh natasha@172.16.238.15
# password: Bl@kW

sudo chown -R natasha:natasha /var/www/html
```

---

## Step 5: Launch the Agent
1. **Manage Jenkins** → **Nodes** → **Storage Server**
2. Click **Launch Agent**
3. Verify status shows **online** 

---

## Step 6: Create Pipeline Job
1. Jenkins → **New Item**
2. Fill in:
```
Name:  datacenter-webapp-job
Type:  Pipeline (NOT Multibranch)
```
3. Click **OK**

---

## Step 7: Add String Parameter
1. Check **This project is parameterized**
2. **Add Parameter** → **String Parameter**
```
Name:          BRANCH
Default Value: master
Description:   Branch to deploy
```

---

## Step 8: Add Pipeline Script
In the **Pipeline** section, paste:

```groovy
pipeline {
    agent {
        label 'ststor01'
    }
    stages {
        stage('Deploy') {
            steps {
                script {
                    if (params.BRANCH == 'master') {
                        sh '''
                            cd /var/www/html
                            git fetch origin
                            git checkout master
                            git pull origin master
                        '''
                    } else if (params.BRANCH == 'feature') {
                        sh '''
                            cd /var/www/html
                            git fetch origin
                            git checkout feature
                            git pull origin feature
                        '''
                    } else {
                        error "Invalid branch: ${params.BRANCH}"
                    }
                }
            }
        }
    }
}
```

4. Click **Save**

---

## Step 9: Run the Pipeline
1. Click **Build with Parameters**
2. Enter `master` or `feature` in BRANCH field
3. Click **Build**
4. Check **Console Output** → should show `Finished: SUCCESS` 

---

## Step 10: Verify
- Click **App** button
- Website loads at `https://<LBR-URL>` 

---

## Summary of Key Fixes
| Issue | Fix |
|---|---|
| Agent permission denied | `sudo chown -R natasha:natasha /var/www/html` |
| Agent offline | Fixed permissions then clicked Launch Agent |
| Conditional deploy | Used `if/else` on `params.BRANCH` in pipeline |
