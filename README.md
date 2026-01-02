# End-To-End-DevOps-CI-CD--PIPELINE

#  EASY_CRUD – End-to-End DevOps CI/CD Project

This project demonstrates a **complete industry-level CI/CD pipeline** using **Jenkins, SonarQube, Docker**, and **GitHub**, deploying a **Spring Boot backend** and **Node/Nginx frontend** automatically.

---

## 🧱 Project Architecture

![Image](https://miro.medium.com/1%2Au3QRAyu7V3tpgdQ3cjVNsg.gif)

![Image](https://miro.medium.com/v2/resize%3Afit%3A2000/1%2AWytnwm9mpIceQ0JLbODMUQ.jpeg)

![Image](https://tomgregory.com/jenkins/sonarqube-quality-gates-in-jenkins-build-pipeline/images/sonarqube-jenkins-interactions.png)

**Flow:**

1. Developer pushes code to GitHub
2. Jenkins pulls code
3. Backend is built using Maven
4. SonarQube performs code quality analysis
5. Quality Gate decides pass/fail
6. Docker images are built
7. Backend & Frontend containers are deployed

---

## 🛠 Tools & Technologies Used

* **Jenkins** – CI/CD automation
* **SonarQube** – Static code analysis
* **Docker** – Containerization
* **GitHub** – Source control
* **Maven** – Build tool
* **Java 17** – Backend runtime
* **Node.js + Nginx** – Frontend build & hosting

---

##  Project Structure

```
EASY_CRUD/
├── backend/
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
└── Jenkinsfile
```

---

##  Step 1: Install Jenkins (Ubuntu)

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Access Jenkins:

```
http://<SERVER-IP>:8080
```

---

##  Step 2: Install Docker on Jenkins Server

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

 This allows Jenkins to run Docker commands.

---

##  Step 3: Install SonarQube (Ubuntu)

```bash
sudo apt install unzip -y
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.2.77730.zip
unzip sonarqube-9.9.2.77730.zip
sudo mv sonarqube-9.9.2.77730 /opt/sonarqube
```

Start SonarQube:

```bash
cd /opt/sonarqube/bin/linux-x86-64
./sonar.sh start
```

Access SonarQube:

```
http://<SONAR-IP>:9000
```

Default login:

```
username: admin
password: admin
```

---

##  Step 4: Configure SonarQube in Jenkins

**Jenkins → Manage Jenkins → System → SonarQube Servers**

* Name: `sonarqube`
* URL: `http://<SONAR-IP>:9000`
* Authentication token: add via Jenkins Credentials
* ✔ Enable “Environment variables”

---

##  Step 5: Configure SonarQube Webhook

**SonarQube → Administration → Configuration → Webhooks → Create**

```
Name: jenkins
URL: http://<JENKINS-IP>:8080/sonarqube-webhook/
```

➡ Required for **Quality Gate** to work.

---

##  Step 6: Backend Dockerfile

 `backend/Dockerfile`

```dockerfile
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app
COPY target/student-registration-backend-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8081
ENTRYPOINT ["java","-jar","app.jar"]
```

---

## \ step 7: Frontend Dockerfile

`frontend/Dockerfile`

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 🧪 Step 8: Jenkins Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any

    stages {

        stage('Pull Code') {
            steps {
                git branch: 'devops',
                    url: 'https://github.com/abhilashmwaghmare/EASY_CRUD.git'
            }
        }

        stage('Backend Build') {
            steps {
                sh '''
                cd backend
                mvn clean package -DskipTests
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    cd backend
                    mvn sonar:sonar \
                    -DskipTests \
                    -Dsonar.projectKey=student-app \
                    -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Backend Docker Deploy') {
            steps {
                sh '''
                cd backend
                docker rm -f backend-app || true
                docker build -t backend-app .
                docker run -d --name backend-app -p 8081:8081 backend-app
                '''
            }
        }

        stage('Frontend Docker Deploy') {
            steps {
                sh '''
                cd frontend
                docker rm -f frontend-app || true
                docker build -t frontend-app .
                docker run -d --name frontend-app -p 80:80 frontend-app
                '''
            }
        }
    }
}
```

---

## 🌐 Step 9: Access the Application

| Service   | URL                       |
| --------- | ------------------------- |
| Frontend  | `http://<SERVER-IP>`      |
| Backend   | `http://<SERVER-IP>:8081` |
| Jenkins   | `http://<SERVER-IP>:8080` |
| SonarQube | `http://<SONAR-IP>:9000`  |

---

## Key Features

*  Automated CI/CD using Jenkins
*  Code quality enforcement with SonarQube Quality Gates
*  Dockerized backend & frontend
*  Zero manual deployment
*  Resume-ready DevOps project

---

##  How to Explain This Project (Interview)

> “I built an end-to-end CI/CD pipeline using Jenkins.
> The pipeline builds a Spring Boot backend, performs static code analysis with SonarQube, enforces Quality Gates, and deploys both backend and frontend as Docker containers.
> Only quality-approved code is deployed automatically.”

---

