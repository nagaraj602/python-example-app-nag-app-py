# python-example-app-nag-app-py
---

# Architecture

```text
GitHub Repo
    |
    v
Jenkins Pipeline
    |
    +--> SonarQube Scan
    |
    +--> Docker Build
    |
    +--> Docker Push (DockerHub)
    |
    +--> Deploy Dev Container
    |
    +--> Manual Approval
    |
    +--> Deploy Stage Container
    |
    +--> Manual Approval
    |
    +--> Deploy Prod Container
```

---

# Tools Required

Install these:

| Tool              | Purpose                |
| ----------------- | ---------------------- |
| Jenkins           | CI/CD                  |
| Docker            | Build & run containers |
| SonarQube         | Code quality           |
| Git               | Source control         |
| DockerHub Account | Push images            |

---

# Directory Structure

Project structure:

```text
python-example-app-nag-app-py/
│
├── Jenkinsfile
├── Dockerfile
├── app.py
├── requirements.txt
└── sonar-project.properties
```

---

# STEP 1 — Create Project Directory

```bash
mkdir python-example-app-nag-app-py
cd python-example-app-nag-app-py
```

---

# STEP 2 — Create Python Application

## File Path

```text
python-example-app-nag-app-py/app.py
```

## Content

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Pipeline Assignment Working"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

# STEP 3 — Create requirements.txt

## File Path

```text
python-example-app-nag-app-py/requirements.txt
```

## Content

```text
flask
```

---

# STEP 4 — Create Dockerfile

## File Path

```text
python-example-app-nag-app-py/Dockerfile
```

## Content

```dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# STEP 5 — Create SonarQube Properties File

## File Path

```text
python-example-app-nag-app-py/sonar-project.properties
```

## Content

```properties
sonar.projectKey=python-example-app-nag-app-py
sonar.projectName=python-example-app-nag-app-py
sonar.sources=.
```

---

# STEP 6 — Push Code to GitHub

Initialize git:

```bash
git init
```

Add files:

```bash
git add .
```

Commit:

```bash
git commit -m "Initial commit"
```

Create GitHub repo and connect:

```bash
git remote add origin https://github.com/yourusername/python-example-app-nag-app-py.git
```

Push:

```bash
git branch -M main
git push -u origin main
```

---

# STEP 7 — Install Docker

Ubuntu:

```bash
sudo apt update

sudo apt install docker.io -y
```

Enable docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Add user to docker group:

```bash
sudo usermod -aG docker $USER
```

Re-login.

Check:

```bash
docker --version
```

---

# STEP 8 — Install Jenkins

## Add Jenkins Repo

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

Install Java:

```bash
sudo apt install fontconfig openjdk-21-jre -y
```

Install Jenkins:

```bash
sudo apt update
sudo apt install jenkins -y
```

Start Jenkins:

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

---

# STEP 9 — Open Jenkins

Open:

```text
http://YOUR_SERVER_IP:8080
```

Get admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Install suggested plugins.

Create admin user.

---

# STEP 10 — Install Required Jenkins Plugins

Go to:

```text
Manage Jenkins → Plugins
```

Install:

| Plugin              |
| ------------------- |
| Docker Pipeline     |
| SonarQube Scanner   |
| Pipeline            |
| Git                 |
| Pipeline Stage View |

Restart Jenkins.

---

# STEP 11 — Install SonarQube Using Docker

Run SonarQube:

```bash
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube:lts-community
```

Open:

```text
http://YOUR_SERVER_IP:9000
```

Default login:

```text
username: admin
password: admin
```

Change password.

---

# STEP 12 — Create SonarQube Token

In SonarQube:

```text
My Account → Security → Generate Token
```

Copy token.

---

# STEP 13 — Configure SonarQube in Jenkins

Go:

```text
Manage Jenkins → System
```

Find:

```text
SonarQube Servers
```

Add:

| Field      | Value                      |
| ---------- | -------------------------- |
| Name       | sonar                      |
| Server URL | http://YOUR_SERVER_IP:9000 |
| Token      | add token                  |

---

# STEP 14 — Install Sonar Scanner

Go:

```text
Manage Jenkins → Tools
```

Find:

```text
SonarQube Scanner
```

Add:

| Name          |
| ------------- |
| sonar-scanner |

Enable auto install.

---

# STEP 15 — Add DockerHub Credentials

Go:

```text
Manage Jenkins → Credentials
```

Add:

| Type     | Username with password |
| -------- | ---------------------- |
| Username | DockerHub username     |
| Password | DockerHub password     |

Credential ID:

```text
dockerhub-creds
```

---

# STEP 16 — Give Jenkins Docker Permission

```bash
sudo usermod -aG docker jenkins
```

Restart Jenkins:

```bash
sudo systemctl restart jenkins
```

---

# STEP 17 — Create Jenkins Pipeline Job

Go:

```text
New Item
```

Select:

```text
Pipeline
```

Name:

```text
python-example-app-nag-app-py
```

---

# STEP 18 — Configure Pipeline

Inside pipeline config:

Select:

```text
Pipeline script from SCM
```

SCM:

```text
Git
```

Repository URL:

```text
https://github.com/yourusername/python-example-app-nag-app-py.git
```

Branch:

```text
*/main
```

Script Path:

```text
Jenkinsfile
```

Save.

---

# STEP 19 — Create Jenkinsfile

## File Path

```text
python-example-app-nag-app-py/Jenkinsfile
```

## Content

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "yourdockerhubusername/pipeline-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    tools {
        sonarQube 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/yourusername/python-example-app-nag-app-py.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    sonar-scanner
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                docker push $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy Dev') {
            steps {
                sh '''
                docker rm -f dev-container || true

                docker run -d \
                  --name dev-container \
                  -p 5001:5000 \
                  $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Approval For Stage') {
            steps {
                input message: 'Deploy to STAGE environment?'
            }
        }

        stage('Deploy Stage') {
            steps {
                sh '''
                docker rm -f stage-container || true

                docker run -d \
                  --name stage-container \
                  -p 5002:5000 \
                  $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Approval For Prod') {
            steps {
                input message: 'Deploy to PROD environment?'
            }
        }

        stage('Deploy Prod') {
            steps {
                sh '''
                docker rm -f prod-container || true

                docker run -d \
                  --name prod-container \
                  -p 5003:5000 \
                  $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
```

---

# STEP 20 — Commit Jenkinsfile

```bash
git add Jenkinsfile
```

```bash
git commit -m "Added Jenkins pipeline"
```

```bash
git push
```

---

# STEP 21 — Run Pipeline

Go Jenkins Job:

```text
Build Now
```

---

# Pipeline Flow

```text
1. Git Checkout
2. SonarQube Scan
3. Docker Build
4. Docker Push
5. Deploy Dev
6. STOP and ask approval
7. Deploy Stage
8. STOP and ask approval
9. Deploy Prod
```

---

# Verify Deployments

## DEV

```text
http://YOUR_SERVER_IP:5001
```

---

## STAGE

```text
http://YOUR_SERVER_IP:5002
```

---

## PROD

```text
http://YOUR_SERVER_IP:5003
```

---

# Important Interview Points

You can explain:

| Topic           | Explanation                  |
| --------------- | ---------------------------- |
| CI              | Continuous Integration       |
| CD              | Continuous Delivery          |
| SonarQube       | Static code analysis         |
| Docker Build    | Container image creation     |
| Docker Push     | Push image to registry       |
| Manual Approval | Human validation gate        |
| Dev/Stage/Prod  | Multi-environment deployment |

---

# Commands to Check Containers

```bash
docker ps
```

---

# View Logs

```bash
docker logs dev-container
```

```bash
docker logs stage-container
```

```bash
docker logs prod-container
```

---

# Stop Containers

```bash
docker stop dev-container
docker stop stage-container
docker stop prod-container
```

---

# Remove Containers

```bash
docker rm dev-container
docker rm stage-container
docker rm prod-container
```

---

# Final Project Structure

```text
python-example-app-nag-app-py/
│
├── app.py
├── requirements.txt
├── Dockerfile
├── sonar-project.properties
└── Jenkinsfile
```
