# Continuous Integration & Continuous Delivery with Jenkins, Nexus, SonarQube, Docker, ECR, ECS, and Slack
This repository documents a complete CI/CD implementation for the **Java web application** using AWS-hosted tooling and Jenkins pipelines.


### End-to-End Flow
1. Developer pushes code to GitHub branch.
2. Jenkins job is triggered (webhook/manual).
3. Jenkins builds and tests code using Maven.
4. Jenkins runs Checkstyle + SonarQube analysis.
5. (CI/CD pipeline) Jenkins waits for Sonar Quality Gate.
6. Jenkins uploads `.war` artifact to Nexus.
7. (Docker pipeline) Jenkins builds Docker image and pushes to ECR.
8. (ECS pipeline) Jenkins forces ECS service redeployment.
9. Slack receives build status notification.

![CI Pipeline Flow](Flow%20of%20Continuous%20Integration%20Pipeline1.png)


It includes:
- Infrastructure setup scripts for Jenkins, Nexus, and SonarQube servers.
- Jenkins pipelines for:
  - CI (build, test, analysis, and Nexus artifact upload)
  - CI + Docker image build/push
  - Full CI/CD (including ECS deployment)
  - Slack notifications
- Architecture and workflow reference images.

---

## 1) High-Level Architecture

### Toolchain
- **Source Control**: GitHub (`vprofile-project`)
- **Orchestrator**: Jenkins (EC2)
- **Build/Test**: Maven + JUnit + Checkstyle
- **Static Analysis**: SonarQube + Quality Gate
- **Artifact Repository**: Nexus 3 (`.war` artifacts)
- **Containerization**: Docker
- **Image Registry**: AWS ECR
- **Deployment**: AWS ECS
- **Notifications**: Slack

---

## 2) AWS Infrastructure Overview

All servers are hosted in the same VPC (`us-east-1`) and communicate over private networking where possible.

- **Jenkins** (EC2): Pipeline execution and integrations.
- **Nexus** (EC2): Stores build artifacts (`.war`).
- **SonarQube** (EC2): Code quality analysis and gate checks.

### Instance Screens

![All Instances](jenkins%2Cnexus%2Csonar%20Instances%20and%20Scripts/All%20the%20instances.png)

![Jenkins Server](jenkins%2Cnexus%2Csonar%20Instances%20and%20Scripts/Jenkins-server.png)

![Nexus Server](jenkins%2Cnexus%2Csonar%20Instances%20and%20Scripts/Nexus-server.png)

![SonarQube Server](jenkins%2Cnexus%2Csonar%20Instances%20and%20Scripts/Sonar-server.png)


---

## 3) Server Setup Scripts

## 3.1 Jenkins Setup
Script: `jenkins,nexus,sonar Instances and Scripts/jenkins-setup.sh`

What it does:
- Installs `openjdk-11-jdk`, `maven`, `wget`, and `unzip`.
- Adds Jenkins APT key and Jenkins Debian repository.
- Installs Jenkins service.

Post-install actions:
1. Open `http://<jenkins-public-ip>:8080`
2. Unlock Jenkins:
	```bash
	sudo cat /var/lib/jenkins/secrets/initialAdminPassword
	```
3. Install required plugins:
	- Git, Maven Integration, Pipeline
	- SonarQube Scanner
	- Nexus Artifact Uploader
	- Docker Pipeline
	- Pipeline: AWS Steps
	- Slack Notification

Configure Jenkins tools:
- JDK: `JDK17`
- Maven: `MAVEN3.9`
- Sonar Scanner: `sonar6.2`

Configure Jenkins credentials:
- `nexuslogin` (Nexus username/password)
- `awscreds` (AWS access key/secret)
- `slacktoken1` (Slack token)

Configure SonarQube server in Jenkins:
- Name: `sonarserver`
- Token-based authentication

## 3.2 Nexus Setup
Script: `jenkins,nexus,sonar Instances and Scripts/nexus-setup.sh`

What it does:
- Installs Java 17 (Corretto) and `wget`.
- Downloads latest Nexus 3 tarball.
- Installs under `/opt/nexus`.
- Creates dedicated `nexus` user.
- Creates and enables `systemd` service.

Post-install actions:
1. Open `http://<nexus-public-ip>:8081`
2. Read initial admin password:
	```bash
	cat /opt/nexus/sonatype-work/nexus3/admin.password
	```
3. Create hosted Maven repository:
	- Type: `maven2 (hosted)`
	- Name: `vprofile-repo`
	- Redeploy: allowed

## 3.3 SonarQube Setup
Script: `jenkins,nexus,sonar Instances and Scripts/sonar-setup.sh`

What it does:
- Applies kernel tuning needed by SonarQube/Elasticsearch.
- Installs Java 17.
- Installs and configures PostgreSQL.
- Creates `sonar` DB user and `sonarqube` database.
- Installs SonarQube 9.9.8.
- Creates SonarQube `systemd` service.
- Configures Nginx reverse proxy.

Post-install actions:
1. Open `http://<sonarqube-public-ip>:9000`
2. Login default: `admin/admin` and change password.
3. Generate token for Jenkins integration.
4. Validate/adjust Quality Gate (default Sonar Way or custom).

---

## 4) Jenkins Pipeline 1: CI + Artifact Upload to Nexus

Pipeline file: `Code Analysis and upload the artifacts/Code Analysis , Upload artifacts.txt`

### Stages
- `Clean Workspace`
- `Fetch code` (branch: `atom`)
- `Build` (`mvn install -DskipTests`)
- `Unit Test` (`mvn test`)
- `Checkstyle Analysis` (`mvn checkstyle:checkstyle`)
- `Sonar Code Analysis`
- `UploadArtifacts` (Nexus uploader)

### Artifact Publishing
- Nexus URL: `172.31.21.29:8081` (private IP)
- Repository: `vprofile-repo`
- Coordinates:
  - `groupId`: `QA`
  - `artifactId`: `vproapp`
  - `type`: `war`
  - file: `target/vprofile-v2.war`

### Screens

![CI Job Configuration](Code%20Analysis%20and%20upload%20the%20artifacts/Screenshot%202025-03-03%20160759.png)

![CI Pipeline Stages](Code%20Analysis%20and%20upload%20the%20artifacts/Screenshot%202025-03-03%20161307.png)

![Nexus Artifact Upload Success](Code%20Analysis%20and%20upload%20the%20artifacts/Screenshot%202025-03-03%20162756.png)

![Artifacts in Nexus Repository](Code%20Analysis%20and%20upload%20the%20artifacts/All%20the%20artifacts%20in%20the%20nexus%20repository.png)

---

## 5) Jenkins Pipeline 2: CI + Docker Build + ECR Push

Pipeline file: `CI for Docker/Pipeline code.txt`

### Additional Environment Variables
- `registryCredential = 'ecr:us-east-1:awscreds'`
- `imageName = '463470955674.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg'`
- `vprofileRegistry = 'https://463470955674.dkr.ecr.us-east-1.amazonaws.com'`

### Additional Stages
- `Quality Gate` (`waitForQualityGate abortPipeline: true`)
- `Build App Image` (Docker build from `./Docker-files/app/multistage/`)
- `Upload App Image` (push build tag + `latest`)
- `Remove Container Images` (cleanup)

### Screens

![Docker in Jenkins](CI%20for%20Docker/Install%20docker%20engine%20in%20jenkins.png)

![Docker Pipeline Run](CI%20for%20Docker/Screenshot%202025-03-03%20173256.png)

![Sonar Quality Gate and Build Flow](CI%20for%20Docker/Screenshot%202025-03-03%20173445.png)

![ECR Push Stage](CI%20for%20Docker/Screenshot%202025-03-03%20173802.png)

![ECR Repository Images](CI%20for%20Docker/Screenshot%202025-03-03%20173943.png)

---

## 6) Jenkins Pipeline 3: Full CI/CD with ECS Deployment

Pipeline file: `Docker CICD/CICD pipeline docker.txt`

### ECS Deployment Stage
Uses AWS CLI from Jenkins:

```bash
aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment
```

Pipeline environment:
- `cluster = vprofile`
- `service = vprofileappsvc`

This triggers a fresh rollout so ECS pulls the updated image tag (commonly `latest` in this setup).

### Screens

![ECS Service Overview](Docker%20CICD/Screenshot%202025-03-03%20174859.png)

![Full CI/CD Job Run](Docker%20CICD/Screenshot%202025-03-03%20175620.png)

![Pipeline Configuration](Docker%20CICD/Screenshot%202025-02-28%20181406.png)

---

## 7) Slack Notification Integration

Pipeline file: `Slack notification/Slack notification.txt`

The pipeline sends a message in `post { always { ... } }` using:
- Channel: `#devopscicd`
- Color map:
  - `SUCCESS -> good`
  - `FAILURE -> danger`
- Message contains job name, build number, and build URL.

### Screens

![Slack Plugin / App Setup](Slack%20notification/Slack%20app%20notification.png)

![Slack Configuration](Slack%20notification/Screenshot%202025-03-03%20170359.png)

![Slack Build Notification](Slack%20notification/Screenshot%202025-03-03%20170407.png)

---

## 8) Required Jenkins Tools, Plugins, and Credentials

## Tools
- `JDK17`
- `MAVEN3.9`
- `sonar6.2`

## Plugins
- Git
- Pipeline
- Maven Integration
- SonarQube Scanner
- Nexus Artifact Uploader
- Docker Pipeline
- Pipeline: AWS Steps
- Slack Notification

## Credentials IDs (used in pipeline code)
- `nexuslogin`
- `awscreds`
- `slacktoken1`

---

## 9) Security Notes (Important)

This repo includes hardcoded passwords in setup scripts (for learning/demo). For production:
- Replace static passwords with secure secrets management.
- Use Jenkins credentials binding for all sensitive values.
- Restrict security groups to least privilege.
- Use private networking and TLS where possible.
- Rotate any exposed credentials immediately.

---

## 10) Quick Start Checklist

1. Launch 3 EC2 instances for Jenkins, Nexus, SonarQube.
2. Run setup scripts from `jenkins,nexus,sonar Instances and Scripts/`.
3. Configure Jenkins tools, plugins, credentials, and SonarQube server.
4. Create `vprofile-repo` in Nexus.
5. Create ECR repo `vprofileappimg` and ECS cluster/service.
6. Create Jenkins pipeline job from one of the pipeline files:
	- CI only: `Code Analysis and upload the artifacts/Code Analysis , Upload artifacts.txt`
	- CI + Docker: `CI for Docker/Pipeline code.txt`
	- Full CI/CD + ECS: `Docker CICD/CICD pipeline docker.txt`
7. Trigger build and verify:
	- Jenkins stages pass
	- Sonar Quality Gate status
	- Nexus artifact upload
	- ECR image tags
	- ECS service redeployment
	- Slack notification delivery

---

## 11) Author Note

This repository serves as a practical DevOps implementation reference for CI/CD on AWS using Jenkins and the Java application lifecycle from source to deployment.

## 🤝 Credits
Source application: vprofile-project by hkhcoder
