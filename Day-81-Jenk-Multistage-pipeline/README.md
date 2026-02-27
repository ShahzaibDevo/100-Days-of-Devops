# Day 81: Jenkins Multistage Pipeline 

---

## Prerequisites
- Jenkins URL: `http://jenkins:8080` — login: `admin` / `Adm!n321`
- Gitea URL: `http://git.stratos.xfusioncorp.com` — login: `sarah` / `Sarah_pass123`
- Storage Server: `ststor01` — user: `natasha` / `Bl@kW`
- Jumphost user: `thor`

---

## STEP 1: Update index.html on Storage Server

SSH into ststor01 from jumphost:
```bash
ssh natasha@ststor01
# password: Bl@kW
```

Clone and update the repo:
```bash
cd /var/www/html
rm -rf web
git clone http://sarah:Sarah_pass123@git.stratos.xfusioncorp.com/sarah/web.git
cd web
echo "Welcome to xFusionCorp Industries" > index.html
git config user.email "sarah@sarah.com"
git config user.name "sarah"
git add index.html
git commit -m "Update index.html"
git push origin master
cp index.html /var/www/html/index.html
cat /var/www/html/index.html
exit
```

Expected output:
```
Welcome to xFusionCorp Industries
```

---

## STEP 2: Verify Gitea Repository

1. Open Gitea browser
2. Login: `sarah` / `Sarah_pass123`
3. Go to `sarah/web` repository
4. Click `index.html`
5. Confirm content shows:
```
Welcome to xFusionCorp Industries
```

---

## STEP 3: Install sshpass on Jenkins

1. Open Jenkins browser
2. Login: `admin` / `Adm!n321`
3. Go to **Manage Jenkins** → **Script Console**
4. Run:

```groovy
def cmd = "which sshpass".execute()
cmd.waitFor()
println cmd.text
println cmd.err.text
```

If sshpass not found, install it:
```groovy
def cmd = ["bash", "-c", "apt-get install -y sshpass 2>&1 || yum install -y sshpass 2>&1"].execute()
cmd.waitFor()
println cmd.text
println cmd.err.text
```

---

## STEP 4: Create Jenkins Pipeline Job

1. Jenkins → **New Item**
2. Name: `deploy-job`
3. Select: **Pipeline** ← NOT Freestyle, NOT Multibranch
4. Click **OK**
5. Scroll to **Pipeline** section
6. Definition: **Pipeline script**
7. Paste this script:

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                sh 'sshpass -p "Bl@kW" ssh -o StrictHostKeyChecking=no natasha@ststor01 "cd /var/www/html/web && git pull origin master && cp /var/www/html/web/index.html /var/www/html/index.html"'
            }
        }
        stage('Test') {
            steps {
                sh '''
                    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://stlb01:8091)
                    [ "$HTTP_CODE" = "200" ] || exit 1
                    curl -s http://stlb01:8091 | grep "Welcome to xFusionCorp Industries" || exit 1
                    echo "Test PASSED"
                '''
            }
        }
    }
}
```

8. Click **Save**

---

## STEP 5: Run the Pipeline

1. Click **Build Now**
2. Click build number in **Build History**
3. Click **Console Output**

Expected output:
```
[Pipeline] stage (Deploy)
Already up to date.
[Pipeline] stage (Test)
Welcome to xFusionCorp Industries
Test PASSED
Finished: SUCCESS
```


---
