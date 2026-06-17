# 🚀 Jenkins CI/CD Assignment – FoodHub & ShopEase Deployment

|  |  |
| --- | --- |
| Name | Abhijeet Satav |
| Course | MCA DevOps |
| Batch | 3 Nov |
| Project | Jenkins CI/CD Automation |

> **Goal:** Deploy FoodHub and ShopEase websites automatically using Jenkins CI/CD on a Linux server.

---

# Part A – MCQ Answers

### 1. What is Jenkins mainly used for?
✅ **Answer:** B) Continuous Integration and Continuous Delivery

---

### 2. Which type of job allows you to define build steps using code in Jenkins?
✅ **Answer:** B) Pipeline Project

---

### 3. Which file is used to define a pipeline in Jenkins?
✅ **Answer:** C) Jenkinsfile

---

### 4. What is the purpose of a Jenkins Agent (Node)?
✅ **Answer:** B) To execute jobs assigned by the Jenkins controller

---

### 5. Which plugin is required to connect Jenkins with GitHub?
✅ **Answer:** B) Git Plugin

---

### 6. What is the purpose of a Webhook in Jenkins CI/CD?
✅ **Answer:** B) To trigger build automatically on code push

---

### 7. Which command is used inside Jenkins Pipeline to execute shell commands?
✅ **Answer:** C) sh

---

### 8. What is the purpose of `post` block in Jenkins Pipeline?
✅ **Answer:** B) Execute steps after pipeline stages

---

### 9. What is the use of `sshagent` in Jenkins Pipeline?
✅ **Answer:** C) Use stored SSH credentials during execution

---

### 10. What happens if a stage fails in Jenkins Pipeline (by default)?
✅ **Answer:** B) The pipeline stops execution

---

# Part B – Jenkins CI/CD Practical Test

## Scenario

Hired as a Junior DevOps Engineer in a startup organization. The development team has already created two static websites and stored their code in GitHub repositories. The task is to set up CI/CD pipelines using Jenkins and automatically deploy both applications on the same target server.

---

# Task 1 – Infrastructure Setup

## Jenkins Server Setup

### Installed Java
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install fontconfig openjdk-21-jre -y
```

### Added Jenkins Repository & Installed Jenkins
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```

### Started Jenkins Service
```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

### Opened Jenkins Port
```bash

sudo ufw allow OpenSSH
sudo ufw enable
```

### Installed Required Plugins
- Git Plugin
- Pipeline Plugin
- SSH Agent Plugin
- GitHub Plugin

### Configured SSH Credentials

Both EC2 instances were created with the **same AWS key pair (.pem file)**, so no new key generation was needed. The public key is already present on both instances automatically.

#### Step 1 – Copy the `.pem` file to Jenkins Server
```bash
# Run this from your local machine
scp -i your-key.pem your-key.pem ubuntu@<JENKINS-IP>:~/.ssh/
```

#### Step 2 – View the private key content
```bash
cat ~/.ssh/your-key.pem
# Copy the entire output
```

#### Step 3 – Add to Jenkins Credentials
```
Manage Jenkins → Credentials → System → Global → Add Credentials
Kind     : SSH Username with private key
ID       : my-ssh-key
Username : ubuntu
Private Key: [Paste the .pem content here]
```

**Screenshot – Jenkins Global Credentials:**

![Global Credentials](Screenshots/Global%20credentials.png)

---


## Target Server Setup

### Installed Nginx
```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

### Opened HTTP Port
```bash
Security Group → Add Inbound Rule → Port 80 → 0.0.0.0/0
Security Group → Add Inbound Rule → Port 22 → 0.0.0.0/0
```

### Created Deployment Directories
```bash
sudo mkdir -p /var/www/html/foodhub
sudo mkdir -p /var/www/html/shopease
```

# Task 2 – Application Repositories

### Application 1: FoodHub Website
- **Repo:** `https://github.com/tanmay23here/FoodHub-website.git`
- **Deploy Path:** `/var/www/html/foodhub`
- **Access URL:** `http://<TARGET-SERVER-IP>/foodhub`

### Application 2: ShopEase Website
- **Repo:** `https://github.com/tanmay23here/shopEase-Website.git`
- **Deploy Path:** `/var/www/html/shopease`
- **Access URL:** `http://<TARGET-SERVER-IP>/shopease`

---

# Task 3 – Jenkins Pipeline Jobs

## 🍕 FoodHub Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any

    environment {
        TARGET_SERVER = "ubuntu@<TARGET-SERVER-IP>"
        DEPLOY_PATH   = "/var/www/html/foodhub"
        REPO_URL      = "https://github.com/tanmay23here/FoodHub-website.git"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: "${REPO_URL}", branch: 'main'
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(['my-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${TARGET_SERVER} \\
                          'sudo mkdir -p ${DEPLOY_PATH} && sudo chown -R ubuntu:ubuntu ${DEPLOY_PATH}'
                        scp -r ./* ${TARGET_SERVER}:${DEPLOY_PATH}/
                    """
                }
            }
        }

        stage('Restart Nginx') {
            steps {
                sshagent(['my-ssh-key']) {
                    sh "ssh -o StrictHostKeyChecking=no ${TARGET_SERVER} 'sudo systemctl restart nginx'"
                }
            }
        }
    }

    post {
        success { echo '✅ FoodHub Deployment Successful!' }
        failure { echo '❌ FoodHub Deployment Failed!' }
    }
}
```

**Screenshots – FoodHub Pipeline:**

![FoodHub Pipeline](Screenshots/FoodHub%20pipeline.png)

![FoodHub Pipeline Build](Screenshots/FoodHub%20pipeline%202.png)

---

## 🛍️ ShopEase Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any

    environment {
        TARGET_SERVER = "ubuntu@<TARGET-SERVER-IP>"
        DEPLOY_PATH   = "/var/www/html/shopease"
        REPO_URL      = "https://github.com/tanmay23here/shopEase-Website.git"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: "${REPO_URL}", branch: 'main'
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(['my-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${TARGET_SERVER} \\
                          'sudo mkdir -p ${DEPLOY_PATH} && sudo chown -R ubuntu:ubuntu ${DEPLOY_PATH}'
                        scp -r ./* ${TARGET_SERVER}:${DEPLOY_PATH}/
                    """
                }
            }
        }

        stage('Restart Nginx') {
            steps {
                sshagent(['my-ssh-key']) {
                    sh "ssh -o StrictHostKeyChecking=no ${TARGET_SERVER} 'sudo systemctl restart nginx'"
                }
            }
        }
    }

    post {
        success { echo '✅ ShopEase Deployment Successful!' }
        failure { echo '❌ ShopEase Deployment Failed!' }
    }
}
```

**Screenshots – ShopEase Pipeline:**

![ShopEase Pipeline](Screenshots/ShopEase%20pipeline.png)

![ShopEase Pipeline Build](Screenshots/ShopEase%20pipeline%202.png)

---

# Task 4 – GitHub Webhook Configuration

Configured GitHub webhooks for both repositories:

- **Payload URL:** `http://<JENKINS-SERVER-IP>:8080/github-webhook/`
- **Content Type:** `application/json`
- **Trigger Event:** Just the Push event.

Enabled in Jenkins:
```
Pipeline → Configure → Build Triggers → ✅ GitHub hook trigger for GITScm polling
```

> Every `git push` now automatically triggers Jenkins and deploys the latest code.

**Screenshots – GitHub Webhooks:**

![FoodHub Webhook](Screenshots/FoodHub%20Webhook.png)

![ShopEase Webhook](Screenshots/ShopEase%20Webhook.png)

---

# Task 5 – Website Modifications

## FoodHub Website

| | Text |
|---|---|
| **Before** | `Fresh Food Everyday` |
| **After** | `Delicious Food Delivered Fast` |

```bash
git add .
git commit -m "updated homepage"
git push origin main
```

**Screenshot – FoodHub Commit History:**

![FoodHub Commit History](Screenshots/FoodHub%20Commit%20History.png)

**Screenshot – FoodHub Live Site:**

![FoodHub Site](Screenshots/FoodHub%20Site.png)

## ShopEase Website

| | Text |
|---|---|
| **Before** | `Welcome to ShopEase` |
| **After** | `Welcome to ShopEase Online Store` |

```bash
git add .
git commit -m "Updated index.html"
git push origin main
```

**Screenshot – ShopEase Commit History:**

![ShopEase Commit History](Screenshots/ShopEase%20Commit%20History.png)

**Screenshot – ShopEase Live Site:**

![ShopEase Site](Screenshots/ShopEase%20Site.png)

---

# Task 6 – Troubleshooting

Issues investigated and resolved during deployment:

| Issue | Resolution |
|---|---|
| SSH key libcrypto error | Regenerated key with `-m PEM` flag |
| Permission denied on SCP | Added `sudo chown` inside pipeline `mkdir` command |
| Placeholder IP in pipeline | Replaced `<TARGET-SERVER-IP>` with actual IP |
| Webhook not triggering | Enabled "GitHub hook trigger for GITScm polling" in Jenkins |

**Screenshots – Target Server Deployment Proof:**

![Target Server Deployment](Screenshots/Target%20Server%20Deplyment.png)

![Target Server Deployment 2](Screenshots/Target%20Server%20Deplyment%202.png)

---

# Bonus Task – Nginx Path-Based Routing

Configured Nginx so both apps run on port 80 via different paths:

```nginx
server {
    listen 80;

    location /foodhub {
        alias /var/www/html/foodhub;
        index index.html;
        try_files $uri $uri/ /foodhub/index.html;
    }

    location /shopease {
        alias /var/www/html/shopease;
        index index.html;
        try_files $uri $uri/ /shopease/index.html;
    }
}
```

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

# CI/CD Architecture

```
Developer (local machine)
        │
        │  git push
        ▼
GitHub Repository
        │
        │  Webhook trigger
        ▼
Jenkins Server (Port 8080)
        │
        │  SSH + SCP
        ▼
Target Server (Nginx)
   ┌────┴────┐
   ▼         ▼
/foodhub  /shopease
```

---

# Project Outcome

✅ Jenkins Installation and Configuration

✅ SSH Credentials with PEM Format (libcrypto compatible)

✅ GitHub Integration with two repositories

✅ GitHub Webhook Configuration (auto-deploy on push)

✅ Two Jenkins Pipelines (FoodHub & ShopEase)

✅ Automated File Deployment via SCP

✅ Nginx Path-Based Routing Configuration

✅ Pipeline Troubleshooting (SSH key, permissions, IP placeholder)

✅ End-to-End CI/CD Automation

---

# Learning Outcomes

Through this project I learned:

- Continuous Integration (CI) and Continuous Delivery (CD)
- Jenkins Installation and Administration
- Jenkins Pipeline as Code (Jenkinsfile / Groovy)
- SSH Key Management (PEM format for compatibility)
- SCP-based File Deployment
- GitHub Webhooks Integration
- Nginx Path-Based Routing
- DevOps Troubleshooting Methodology
- Multi-Application Deployment on Single Server

---
