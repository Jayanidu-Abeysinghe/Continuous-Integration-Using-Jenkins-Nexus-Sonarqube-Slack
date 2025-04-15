# 🚀 Continuous Integration Using Jenkins, Nexus, SonarQube & Slack

![Architecture Diagram](Flow of Continuous Integration Pipeline.png.webp)


This project demonstrates a complete CI/CD pipeline for automating the **build**, **test**, **code quality analysis**, **artifact management**, and **team notifications** using:

- 🛠️ Jenkins for CI/CD automation  
- 📦 Nexus for artifact repository  
- 🔍 SonarQube for static code analysis  
- 🐳 Docker for containerization  
- 🔔 Slack for real-time notification  

---


## 📌 Project Objective

To set up a full **CI/CD pipeline** that automates:

1. **Code integration & build** with Jenkins
2. **Code quality checks** with SonarQube
3. **Artifact creation** and upload to Nexus
4. **Docker image creation & deployment**
5. **Slack notifications** on pipeline status

---

## 🛠️ Tools & Technologies

| Tool         | Purpose                            |
|--------------|-------------------------------------|
| Jenkins      | Build and automation server         |
| SonarQube    | Static code analysis                |
| Nexus        | Artifact storage                    |
| Docker       | Containerization                    |
| Slack        | Notification platform               |

---

## ⚙️ Step-by-Step Pipeline Setup

### 1️⃣ Jenkins Setup

- Launch Jenkins using a VM or Docker.
- Install required plugins:  
  - Git
  - Docker Pipeline
  - SonarQube Scanner
  - Nexus Artifact Uploader
  - Slack Notification

**Configure Jenkins System Settings** for:
- SonarQube Server
- Slack Webhook
- Nexus Repository URL

> Scripts to automate this:  
`jenkins,nexus,sonar Instances and Scripts/jenkins-setup.sh`

---

### 2️⃣ SonarQube Setup

- Start a SonarQube server instance.
- Create a project token and configure it in Jenkins.
- Define SonarQube quality gates.

> Setup Script:  
`jenkins,nexus,sonar Instances and Scripts/sonar-setup.sh`

---

### 3️⃣ Nexus Repository Setup

- Start a Nexus instance and create a private hosted Maven repository.
- Use this as the target for artifact uploads.

> Setup Script:  
`jenkins,nexus,sonar Instances and Scripts/nexus-setup.sh`

---

### 4️⃣ Pipeline Flow

![Pipeline Flow](Flow%20of%20Continuous%20Integration%20Pipeline.png)

---

### 5️⃣ Jenkins Pipeline Script Overview

1. **Checkout Code**
2. **Static Code Analysis (SonarQube)**
3. **Build Project & Create Artifacts**
4. **Upload Artifacts to Nexus**
5. **Build Docker Image & Push to Registry**
6. **Send Slack Notification**

Scripts and pipelines are available under:
- `CI for Docker/`
- `Docker CICD/`
- `Code Analysis and upload the artifacts/`
- `Slack notification/`

---

## ✅ Slack Notifications

- Integrate your Jenkins pipeline with Slack using a webhook.
- Send success/failure messages to your DevOps channel.

---



