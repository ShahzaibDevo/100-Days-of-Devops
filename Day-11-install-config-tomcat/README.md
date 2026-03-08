# Day 11: Install and Configure Apache Tomcat Server 🚀

## Task Overview
Install and configure **Apache Tomcat** on **App Server 2 (`stapp02`)** in Stratos DC. The server must listen on port `5000`, and a pre-built `ROOT.war` from the Jump host must be deployed so the application is accessible directly at `http://stapp02:5000`.

---

## 📋 Requirements

| Requirement | Details |
|-------------|---------|
| Server | `stapp02` (App Server 2) |
| Tomcat Port | `5000` |
| WAR Source | `/tmp/ROOT.war` on Jump host |
| WAR Destination | `/opt/tomcat/webapps/` on `stapp02` |
| Access URL | `http://stapp02:5000` |

---

## 🛠️ Solution

### Step 1: Install Java & Create Tomcat User

SSH into `stapp02` and install Java:

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install -y openjdk-11-jdk wget tar

# RHEL/CentOS
sudo yum install -y java-11-openjdk-devel wget tar
```

Create a dedicated user and group for Tomcat:

```bash
sudo groupadd tomcat || true
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat || true
```

---

### Step 2: Download & Install Tomcat

```bash
cd /tmp
TOMCAT_VER=9.0.75
wget https://downloads.apache.org/tomcat/tomcat-9/v${TOMCAT_VER}/bin/apache-tomcat-${TOMCAT_VER}.tar.gz

sudo mkdir -p /opt/tomcat
sudo tar xzf apache-tomcat-${TOMCAT_VER}.tar.gz -C /opt/tomcat --strip-components=1
sudo chown -R tomcat:tomcat /opt/tomcat
sudo chmod -R u+rX /opt/tomcat
```

---

### Step 3: Configure Port 5000

Change the default HTTP connector port from `8080` to `5000` in `server.xml`:

```bash
sudo sed -i '0,/Connector port="8080"/s//Connector port="5000"/' /opt/tomcat/conf/server.xml
```

Verify the change:

```bash
grep -n 'Connector.*port="5000"' /opt/tomcat/conf/server.xml
```

---

### Step 4: Deploy ROOT.war from Jump Host

**Run on the Jump host:**

```bash
# Copy WAR to stapp02
scp /tmp/ROOT.war <your-ssh-user>@stapp02:/tmp/

# Move it into Tomcat webapps
ssh <your-ssh-user>@stapp02 'sudo mv /tmp/ROOT.war /opt/tomcat/webapps/ && sudo chown tomcat:tomcat /opt/tomcat/webapps/ROOT.war'
```

> Tomcat will automatically extract `ROOT.war` to `/opt/tomcat/webapps/ROOT` on startup.

---

### Step 5: Create systemd Service

```bash
sudo tee /etc/systemd/system/tomcat.service > /dev/null <<'EOF'
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat
Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now tomcat
```

> If your Java path differs, find it using: `readlink -f /usr/bin/java`

---

### Step 6: Open Firewall Port 5000

```bash
# Debian/Ubuntu (ufw)
sudo ufw allow 5000/tcp && sudo ufw reload

# RHEL/CentOS (firewalld)
sudo firewall-cmd --add-port=5000/tcp --permanent && sudo firewall-cmd --reload
```

---

### Step 7: Verify Deployment

Check Tomcat logs:

```bash
sudo tail -n 200 /opt/tomcat/logs/catalina.out
```

Test the endpoint:

```bash
curl -I http://stapp02:5000
```

Expected output:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
```

---

## ✅ Verification Output

```
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Transfer-Encoding: chunked
Server: Apache-Coyote/1.1
```

---

## 💡 Key Concepts Learned

**Apache Tomcat** is a Java Servlet container used to deploy `.war` (Web Application Archive) files. Key concepts covered in this task:

- **`server.xml`** — Main Tomcat config file; the `<Connector>` element defines the listening port.
- **`webapps/ROOT`** — The default context path. Deploying `ROOT.war` makes the app accessible at `/` (root URL).
- **`systemd` service** — Manages Tomcat as a system service for auto-start on boot.
- **`JAVA_HOME`** — Must point to a valid JDK/JRE installation for Tomcat to start.
- **WAR auto-deployment** — Tomcat automatically unpacks `.war` files placed in the `webapps/` directory.

---
