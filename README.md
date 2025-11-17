#  Complete CI/CD Pipeline – Jenkins, SonarQube, Nexus, Docker, DockerHub, Tomcat Deployment

This repository demonstrates a **full end-to-end DevOps CI/CD pipeline** for a Java Web Application (WAR) deployed on **Tomcat** using Docker.
Pipeline includes:

✔ Jenkins (running inside Docker)
✔ SonarQube (static code analysis)
✔ Nexus Repository (artifact storage)
✔ Docker (image building & execution)
✔ DockerHub (image registry)
✔ Tomcat (runtime deployment)
✔ GitHub (source code)

Everything is deployed on an **AWS EC2 instance**.

---

# 🏗️ **1. EC2 Instance Setup**

### ✔ Instance Type: `t2.large`

### ✔ OS: Ubuntu 22.04

### ✔ Minimum Storage: 12GB

### ✔ Open Ports:

| Port | Usage                    |
| ---- | ------------------------ |
| 22   | SSH                      |
| 8080 | Jenkins                  |
| 8081 | Nexus                    |
| 9000 | SonarQube                |
| 8080 | Tomcat App Deployment    |
| 8082 | loginregistration        |
---
<img width="620" height="232" alt="1" src="https://github.com/user-attachments/assets/66c5ce5d-c21a-4344-9287-bb6914e647ef" />


# 🐳 **2. Install Docker on EC2**

```bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```

Logout & login again.

---

# 🎛️ **3. Run SonarQube in Docker**

SonarQube requires good RAM (2GB+).

```bash
docker run -dt --name sonar \
  -p 9000:9000 \
  sonarqube:lts
```

Access:

```
http://<EC2_PUBLIC_IP>:9000
```

---

# 🧰 **4. Run Nexus Repository**

```bash
docker run -dt --name nexus \
  -p 8081:8081 \
  sonatype/nexus3
```

Access:

```
http://<EC2_PUBLIC_IP>:8081
```

Retrieve initial password:

```bash
docker exec -it nexus cat /nexus-data/admin.password
```

---

# 🛠️ **5. Build Custom Jenkins Image with Docker Installed**

### Create a directory:

```bash
mkdir jenkins-docker
cd jenkins-docker
```

### Add **Dockerfile**:

```dockerfile
FROM jenkins/jenkins:lts

USER root

RUN apt-get update && \
    apt-get install -y docker.io

RUN groupadd -g 999 docker || true
RUN usermod -aG docker jenkins

USER jenkins
```

### Build the image:

```bash
docker build -t jenkins-docker:latest .
```

---

# 🧩 **6. Run Jenkins Container**

```bash
docker run -dt \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins-docker:latest
```

Retrieve initial password:

```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Access:

```
http://<EC2_PUBLIC_IP>:8080
```

---

# 🔧 **7. Jenkins Configuration**

### Install Plugins:

* Git plugin
* Pipeline plugin
* SonarQube Scanner
* Docker Pipeline
* Nexus Artifact Uploader (optional)
* Docker pipeline

### Configure Tools:

**Manage Jenkins → Global Tool Configuration**

#### Maven

* Name: `Maven`
* Install automatically

---

# 🔐 **8. Add Required Jenkins Credentials**

| ID             | Type        | Usage                   |
| -------------- | ----------- | ----------------------- |
| sonar          | Secret Text | SonarQube token         |
| nexus          | User/Pass   | Nexus admin credentials |
| docker-hub     | User/Pass   | DockerHub login         |

---

# 🗳️ **9. Update `pom.xml` for Nexus (distributionManagement)**

```xml
<distributionManagement>
    <repository>
        <id>nexus</id>
        <url>http://<EC2_PUBLIC_IP>:8081/repository/maven-releases/</url>
    </repository>
</distributionManagement>
```

---

# 🔧 **10. Jenkinsfile (CI/CD Pipeline Script)**

```groovy
pipeline {
    agent any

    environment {
        SONARQUBE_URL = 'http://sonar:9000'
        NEXUS_URL     = 'http://nexus:8081'
        DOCKER_IMAGE  = "rakesh268/loginregistration"
        VERSION       = "${env.BUILD_NUMBER}"
    }

    tools {
        maven 'Maven-3'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
            post {
                always {
                    junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar', variable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=loginregistration \
                        -Dsonar.host.url=${SONARQUBE_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {

                    sh '''
                    echo "<settings>
                            <servers>
                                <server>
                                    <id>nexus</id>
                                    <username>${NEXUS_USER}</username>
                                    <password>${NEXUS_PASS}</password>
                                </server>
                            </servers>
                        </settings>" > settings.xml
                    '''

                    sh 'mvn deploy -DskipTests -s settings.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${VERSION}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh """
                        echo ${DH_PASS} | docker login -u ${DH_USER} --password-stdin
                        docker push ${DOCKER_IMAGE}:${VERSION}
                        docker tag ${DOCKER_IMAGE}:${VERSION} ${DOCKER_IMAGE}:latest
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

    }

    post {
        always {
            cleanWs()
        }
    }
}
```

---

# 🐳 **11. Tomcat Dockerfile (App Deployment)**

```dockerfile
FROM tomcat:9-jdk17

RUN rm -rf /usr/local/tomcat/webapps/*

COPY target/loginregistration.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
```

---

# 🚀 **12. Deploy Application Container**

```bash
docker run -d -p 8082:8080 --name bookstore login  rakesh268/loginregistration:latest
```

Open in browser:

```
http://<EC2_PUBLIC_IP>:8082
```

---

# 🎉 **Final Outcome**

You now have a **complete CI/CD pipeline**:

✔ GitHub → Jenkins → SonarQube → Nexus → Docker → DockerHub → Tomcat

✔ Fully automated WAR build
✔ Static code analysis
✔ Artifact upload
✔ Docker image build
✔ Push to DockerHub
✔ Deployment in Tomcat container

This is **production-grade DevOps pipeline**.

---

