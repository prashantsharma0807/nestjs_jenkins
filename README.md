# NestJS CI/CD Deployment using Jenkins, Docker and AWS EC2

## Project Information

Repository:
https://github.com/prashantsharma0807/nestjs_jenkins

Branch:
main

Architecture:

GitHub Repository
↓
Jenkins (Docker Container)
↓
Docker Build
↓
NestJS Docker Container
↓
EC2 Instance

---

# 1. Create EC2 Instance

Launch Ubuntu EC2 Instance.

Security Group Inbound Rules:

* SSH (22)
* HTTP (80)
* HTTPS (443)
* Jenkins (8080)
* Application (3000)

---

# 2. Connect to EC2

```bash
chmod 400 key.pem
ssh -i key.pem ubuntu@<EC2_PUBLIC_IP>
```

Verification:

```bash
whoami
```

Expected:

```text
ubuntu
```

---

# 3. Install Java

Jenkins requires Java.

```bash
sudo apt update -y
sudo apt install openjdk-21-jdk -y
```

Verification:

```bash
java -version
```

Expected:

```text
openjdk version "21"
```

---

# 4. Install Docker

```bash
sudo apt install docker.io -y
```

Start Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Verification:

```bash
docker --version
```

Expected:

```text
Docker version ...
```

Check Service:

```bash
sudo systemctl status docker
```

Expected:

```text
active (running)
```

---

# 5. Install Git

```bash
sudo apt install git -y
```

Verification:

```bash
git --version
```

Expected:

```text
git version ...
```

---

# 6. Create Custom Jenkins Image

Create folder:

```bash
mkdir jenkins-docker
cd jenkins-docker
```

Create Dockerfile:

```dockerfile
FROM jenkins/jenkins:lts-jdk21

USER root

RUN apt-get update && \
    apt-get install -y docker.io

USER jenkins
```

Build Image:

```bash
docker build -t jenkins-docker .
```

Verification:

```bash
docker images
```

Expected:

```text
jenkins-docker
```

---

# 7. Run Jenkins Container

```bash
docker run -d \
  --name jenkins \
  -u root \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins-docker
```

Verification:

```bash
docker ps
```

Expected:

```text
jenkins
Up
0.0.0.0:8080->8080
```

---

# 8. Verify Docker Inside Jenkins

Enter Container:

```bash
docker exec -it jenkins bash
```

Verify Docker:

```bash
docker --version
```

Expected:

```text
Docker version ...
```

Verify Socket Access:

```bash
docker ps
```

Expected:

List of host containers.

Exit:

```bash
exit
```

---

# 9. Open Jenkins

Open Browser:

http://<EC2_PUBLIC_IP>:8080

Verification:

Jenkins Unlock Screen appears.

---

# 10. Get Initial Password

```bash
docker logs jenkins
```

Copy password and unlock Jenkins.

Verification:

Jenkins Dashboard opens successfully.

---

# 11. Install Jenkins Plugins

Install:

* Git Plugin
* Pipeline Plugin
* Docker Pipeline Plugin
* GitHub Integration Plugin

Verification:

Manage Jenkins → Plugins

Installed plugins should be visible.

---

# 12. Configure Docker Permission

```bash
sudo chmod 666 /var/run/docker.sock
```

Verification:

```bash
ls -l /var/run/docker.sock
```

Expected:

```text
srw-rw-rw-
```

---

# 13. Create Pipeline Job

Dashboard → New Item

Select:

Pipeline

Name:

NestJS-Deploy

---

# 14. Configure SCM

Pipeline Definition:

Pipeline script from SCM

SCM:

Git

Repository URL:

https://github.com/prashantsharma0807/nestjs_jenkins.git

Branch:

*/main

Script Path:

Jenkinsfile

Save.

Verification:

Click Build Now.

Jenkins should successfully clone repository.

---

# 15. Dockerfile

Project contains:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["node", "dist/main"]
```

---

# 16. Jenkinsfile

Project contains:

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = "nestjs-app"
    }

    stages {

        stage('Build') {
            steps {
                sh 'docker build -t $APP_NAME .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh 'docker stop $APP_NAME || true'
                sh 'docker rm $APP_NAME || true'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d -p 3000:3000 --name $APP_NAME $APP_NAME'
            }
        }
    }
}
```

---

# 17. Run Pipeline

Click:

Build Now

Verification:

Build Console Output ends with:

```text
Finished: SUCCESS
```

---

# 18. Verify Application Container

```bash
docker ps
```

Expected:

```text
jenkins
nestjs-app
```

---

# 19. Verify Application

Browser:

http://<EC2_PUBLIC_IP>:3000

Expected:

NestJS application loads successfully.

---

# 20. Configure GitHub Webhook (Optional)

GitHub Repository:

Settings → Webhooks

Add Webhook

Payload URL:

http://<EC2_PUBLIC_IP>:8080/github-webhook/

Content Type:

application/json

Events:

Just the push event

---

# 21. Enable Jenkins Trigger

Job Configuration

Build Triggers:

✓ GitHub hook trigger for GITScm polling

Save.

---

# 22. Verify Auto Deployment

Make a code change.

Push to GitHub:

```bash
git add .
git commit -m "Webhook Test"
git push origin main
```

Verification:

Jenkins automatically starts a new build without clicking Build Now.

---

# Final Verification Commands

Check Jenkins Container:

```bash
docker ps
```

Check Jenkins Logs:

```bash
docker logs jenkins
```

Check Application Container:

```bash
docker ps
```

Check Application Logs:

```bash
docker logs nestjs-app
```

Check Application:

http://<EC2_PUBLIC_IP>:3000

Check Jenkins:

http://<EC2_PUBLIC_IP>:8080

```
```
