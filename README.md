# 🚀 Jenkins CI/CD Pipeline — .NET 8 Application Deployment on IIS (Windows)

This project demonstrates a complete **CI/CD pipeline implementation** for a .NET (ASP.NET Core 8) application using:

- Jenkins (Linux EC2 Controller)
- SonarQube (Code Quality Analysis)
- OWASP Dependency Check (Security Scanning)
- Windows Jenkins Agent
- IIS (Application Hosting)

Repository:  
https://github.com/thegagankapoor/eShopOnWeb

---

# 🏗 Architecture Overview

| Component | Purpose |
|-----------|----------|
| Jenkins (Linux EC2) | CI/CD Controller |
| SonarQube | Code Quality & Static Analysis |
| OWASP Dependency Check | Vulnerability Scanning |
| Windows Agent | Deployment Executor |
| IIS | Application Hosting |
| GitHub | Source Code Repository |

---

# 📌 Prerequisites

- AWS EC2 Instance (Ubuntu)
- Windows Machine (for IIS & Agent)
- GitHub Account
- Open Ports: 8080, 9000, 50000

---

# ⚙️ Infrastructure Provisioning (Automated)

Jenkins and SonarQube are provisioned automatically using an **EC2 User Data Script (available in this repository).**

The script performs:

- Install Java 21
- Install Jenkins
- Install SonarQube
- Configure required ports (8080, 9000)
- Start Jenkins & SonarQube services

No manual installation is required.

---

# 🔓 Access Jenkins (First-Time Setup)

Open in browser:

```
http://<EC2-Public-IP>:8080
```

### 🔑 Get Initial Admin Password

Run on EC2 instance:

```
sudo cat /var/lib/jenkins/.jenkins/secrets/initialAdminPassword
```

Copy the password and paste it into Jenkins UI to unlock.

---

# 📊 Access SonarQube

Open in browser:

```
http://<EC2-Public-IP>:9000
```

---

## 🔑 How to Generate SonarQube Token

1. Open SonarQube in browser:

   ```
   http://<EC2-Public-IP>:9000
   ```

2. Log in (default credentials if first time):

   - Username: `admin`
   - Password: `admin`

   ⚠️ You will be prompted to change the password after first login.

3. Click on your profile icon (top right) → **My Account**

4. Go to:
   **Security → Generate Tokens**

5. Configure:

   - Token Name: `jenkins-token`
   - Token Type: Global Analysis Token

6. Click **Generate**

7. Copy the generated token immediately (it will not be shown again).

Store it securely.

---

# 🔌 Jenkins Configuration

## 1️⃣ Install Required Plugins

Go to:

Manage Jenkins → Plugins → Available Plugins

Install:

- .NET SDK Support
- OWASP Dependency-Check
- SonarQube Scanner
- Pipeline Stage View

Restart Jenkins after installation.

---

## 2️⃣ Configure Tools

Go to:

Manage Jenkins → Tools

### 🔹 .NET SDK Installation

* Name: `dotnet-8`
* Install automatically: ✅
* .NET Version: `.NET 8`
* Release: 8.0.24
* SDK: 8.0.418
* Platform: `Linux x64`
---

### 🔹 OWASP Dependency-Check

- Name: `DP-Check`
- Install automatically: ✅
- Install from GitHub

---

### 🔹 SonarQube Scanner

- Name: `SonarQube`
- Install automatically: ✅

---

## 🔒 Add Credentials in Jenkins

Go to:

Manage Jenkins → Credentials → System → Global Credentials → Add Credentials

### SonarQube Token

- Kind: Secret text
- ID: `sonarqube-token`
- Secret: SonarQube Token

---

# 🔗 Configure SonarQube in Jenkins

Go to:

Manage Jenkins → System → SonarQube Installations

Add:

- Name: `SonarQube`
- Server URL: `http://Sonar_URL:9000`
- Server Authentication Token: `sonarqube-token`

Save.

---

# 🖥 Windows Machine Setup (Deployment Server)

This machine acts as:

- Jenkins Agent
- IIS Web Server

---

## 1️⃣ Install IIS

Press `Windows + R`

Type:

```
optionalfeatures
```

Enable:

- Internet Information Services
- IIS Management Console
- ISAPI Extensions
- ISAPI Filters
- Static Content
- Default Document
- Directory Browsing
- HTTP Errors

Verify:

```
http://localhost
```

---

> ⚠️ Run PowerShell as Administrator for the following steps.

---

## 2️⃣ Install Java (Required for Jenkins Agent)

Install Java 17.

Verify:

```
java -version
```

---

## 3️⃣ Install ASP.NET Core Hosting Bundle (.NET 8)

Install ASP.NET Core Hosting Bundle (.NET 8) – Windows x64.

After installation:

```
iisreset
```

Verify:

```
dotnet --list-runtimes
```

---

## ❌ dotnet command not found (Windows)

If this fails:

```
dotnet --list-runtimes
```

or you see:

```
'dotnet' is not recognized as an internal or external command
```

### 🔎 Reason

.NET SDK or Runtime is not installed, or PATH is not configured.

---

### ✅ Fix: Install .NET 8 SDK

1. Visit:
   https://dotnet.microsoft.com/download/dotnet/8.0
2. Download:
   .NET SDK 8.x (Windows x64)
3. Install the SDK.
4. Restart the machine.
5. Verify:

```
dotnet --version
dotnet --list-runtimes
```

---

# 🤖 Create Windows Jenkins Agent

Go to:

Manage Jenkins → Nodes → New Node

Create:

- Name: `windows-iis-agent`
- Type: Permanent Agent
- Executors: 1
- Remote root directory: `C:\Jenkins`
- Labels: `windows-iis-agent`
- Usage: Only build jobs with matching label
- Launch Method: Launch agent by connecting it to the controller
- Click on "Save"

Save.

---

## 🔗 Connect Windows Agent

On Windows machine:

```
mkdir C:\Jenkins
cd C:\Jenkins
```

After creating the node:

1. Click on `windows-iis-agent`.
2. Scroll to **Run from agent command line (Windows)**.
3. Copy the provided commands.
4. Execute them in PowerShell (Run as Administrator).

---

# 🛠 Agent Connection Fix (If Offline)

## Enable TCP Port for Inbound Agents

Go to:

Manage Jenkins → Security → Agents

If **TCP port for inbound agents** is:

```
Disabled
```

Change it to:

```
Fixed
```

Enter:

```
50000
```

Save.

---

## Allow Port in EC2 Security Group

Add inbound rule:

- Type: Custom TCP
- Port: 50000
- Source: Your IP

---

# 📦 Create Jenkins Pipeline Job

New Item → Pipeline

Name:

```
eShopOnWeb-Pipeline
```

Configure:

- Pipeline script from SCM → Git
- Repository: https://github.com/thegagankapoor/eShopOnWeb
- Branch: main
- Script Path: Jenkinsfile

Save.

---

# 🔁 Pipeline Flow

1. Clone repository from GitHub  
2. Restore & Build .NET Application  
3. Run Unit Tests  
4. Run OWASP Dependency Check  
5. Run SonarQube Analysis  
6. Publish application  
7. Deploy to IIS via Windows Agent  

---

# 🌍 Access Application

After successful deployment:

```
http://localhost:8081
```

---

# ⚠️ Common Errors

## SQL Server Connection Error

Example:

```
SqlException: error 26 - Error Locating Server/Instance Specified
```

Reason: Database not configured.

Fix: Update connection string properly.

---

# 🎯 DevOps Concepts Demonstrated

- CI/CD Pipeline Implementation  
- Infrastructure Automation (User Data)  
- Multi-node Jenkins Setup  
- Static Code Analysis  
- Security Scanning  
- Secure Credential Management  
- Cross-Platform Deployment  
- IIS Hosting  

---

# 👨‍💻 Author

**Gagan Kapoor**  
DevOps Enthusiast | Cloud & CI/CD Practitioner