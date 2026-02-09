

# Day 77: Jenkins Deploy Pipeline

## 🎯 Lab Objective

Deploy a static website from a **Gitea repository** to **Nautilus App Servers** using a **Jenkins pipeline**.

---

## 🖥 Environment Setup

* **Jenkins Server**

  * URL: `http://<Jenkins-URL>`
  * Username: `admin`
  * Password: `Adm!n321`

* **Gitea Server**

  * URL: `http://<Gitea-IP>:3000`
  * Username: `sarah`
  * Password: `Sarah_pass123`
  * Repository: `web_app` (already cloned on Storage Server under `/var/www/html`)

* **Storage Server**

  * Slave node labeled `ststor01`
  * Remote root directory: `/var/www/html`

* **Web Server**

  * Apache installed on all app servers
  * Listening on port `8080`

---

## 🔧 Jenkins Pipeline Configuration

1. **Create Pipeline Job**

   * Job Name: `datacenter-webapp-job`
   * Type: **Pipeline** (not Multibranch)

2. **Configure Node**

   * Run job on **Storage Server** (slave node)
   * Workspace: `/var/www/html/workspace/datacenter-webapp-job`

3. **Pipeline Script Example**

```groovy
pipeline {
    agent { label 'ststor01' }
    stages {
        stage('Deploy') {
            steps {
                git credentialsId: 'gitea-sarah-creds', 
                    url: 'http://git.stratos.xfusioncorp.com/sarah/web_app.git', 
                    branch: 'master'
                
                sh 'cp -r * /var/www/html/'
                
                sh '''
                sudo systemctl restart apache2 || \
                sudo systemctl restart httpd || \
                sudo systemctl restart nginx
                '''
            }
        }
    }
}
```

> ⚠️ Notes:
>
> * Ensure Jenkins user has **passwordless sudo** for restart commands.
> * Git credentials must be configured in Jenkins (`gitea-sarah-creds`).
> * Use the correct branch from Gitea (`master`).

---

## ✅ Final Outcome

* Pipeline successfully fetched code from **Gitea**
* Static website deployed to `/var/www/html` on Storage Server
* Apache service restarted
* Changes visible at main URL (`https://<LBR-URL>`)
