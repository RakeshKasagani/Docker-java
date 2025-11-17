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
<img width="608" height="338" alt="2" src="https://github.com/user-attachments/assets/cf0a81b0-d0ed-42e6-a4af-2ccdeb72f3db" />
<img width="643" height="211" alt="3" src="https://github.com/user-attachments/assets/132ceac6-49cb-42d2-a043-e95a78d7b022" />
<img width="461" height="344" alt="4" src="https://github.com/user-attachments/assets/81bb30a1-cea6-4df6-8cd3-5c2caa49a3ed" />
<img width="621" height="222" alt="5" src="https://github.com/user-attachments/assets/4f096fc9-3758-410c-9b53-b2bfc0e1d941" />
<img width="604" height="339" alt="6" src="https://github.com/user-attachments/assets/b6976880-b9a6-408d-8c04-9fa45220378c" />
<img width="618" height="278" alt="7" src="https://github.com/user-attachments/assets/c6d512b5-fa85-4697-bf04-29017e4f62b9" />
<img width="618" height="278" alt="7" src="https://github.com/user-attachments/assets/bde81a58-815c-428c-9e18-7fe6a761c38f" />

# 🐳 **2. Install Docker on EC2**

```bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```
<img width="476" height="62" alt="11" src="https://github.com/user-attachments/assets/97c10b31-0e81-45f1-b67f-977fe301503b" />
<img width="680" height="141" alt="12" src="https://github.com/user-attachments/assets/7f88ced0-fefb-44f4-a0de-592ccfa0fb37" />
<img width="542" height="110" alt="13" src="https://github.com/user-attachments/assets/b560da64-0188-47dc-9c53-a75ca087fb67" />
<img width="899" height="250" alt="14" src="https://github.com/user-attachments/assets/a1a8a220-a4fd-4f44-8d31-e373607dbb8f" />

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
<img width="567" height="227" alt="15" src="https://github.com/user-attachments/assets/d6507875-072f-4923-b056-02cd66ecc6e7" />
<img width="959" height="432" alt="17" src="https://github.com/user-attachments/assets/2bb64a20-eef1-4c96-9007-03ec9172c427" />



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
<img width="584" height="271" alt="16" src="https://github.com/user-attachments/assets/8628fbc0-c592-45c0-9884-39c540686d71" />
<img width="946" height="457" alt="18" src="https://github.com/user-attachments/assets/32b9219a-51a0-4e7c-8d08-d9a53916ffb7" />
<img width="953" height="406" alt="19" src="https://github.com/user-attachments/assets/9b2a9008-8437-4e77-b542-4ebc0462f6d0" />

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
<img width="611" height="179" alt="20" src="https://github.com/user-attachments/assets/3f67cded-b300-4ccd-8b12-d0ba0d4b08c7" />

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
<img width="952" height="269" alt="22" src="https://github.com/user-attachments/assets/ed645812-6119-4b17-a062-cfdd1ba1f7d0" />
<img width="902" height="457" alt="23" src="https://github.com/user-attachments/assets/c5f4c040-975c-4117-a78f-2c5944eb4023" />
<img width="923" height="460" alt="24" src="https://github.com/user-attachments/assets/abcbf46b-0e86-4f8d-9bd9-209c967267f4" />



---

# 🔧 **7. Jenkins Configuration**

### Install Plugins:

* Git plugin
* Pipeline plugin
* SonarQube Scanner
* Docker Pipeline
* Nexus Artifact Uploader (optional)
* Docker pipeline
  
  <img width="915" height="459" alt="32" src="https://github.com/user-attachments/assets/620f8443-e961-4f6c-9c38-22647b32254b" />

  

### Configure Tools:

**Manage Jenkins → Global Tool Configuration**

#### Maven

* Name: `Maven`
* Install automatically
  
<img width="926" height="437" alt="25" src="https://github.com/user-attachments/assets/580cbac7-3af1-4595-85a1-36edbb64faa6" />

---

# 🔐 **8. Add Required Jenkins Credentials**

| ID             | Type        | Usage                   |
| -------------- | ----------- | ----------------------- |
| sonar          | Secret Text | SonarQube token         |
| nexus          | User/Pass   | Nexus admin credentials |
| docker-hub     | User/Pass   | DockerHub login         |

---
<img width="931" height="429" alt="27" src="https://github.com/user-attachments/assets/205d7525-f010-4a8d-b755-55b647638a65" />
<img width="932" height="472" alt="28" src="https://github.com/user-attachments/assets/f7332a19-5347-4a51-837b-9177aecf44d4" />
<img width="926" height="457" alt="29" src="https://github.com/user-attachments/assets/52cf7b43-2c01-475b-aeb7-85e32f6c7f6f" />




# 🗳️ **9. Update `pom.xml` for Nexus (distributionManagement)**

```xml
<distributionManagement>
    <repository>
        <id>nexus</id>
        <url>http://<EC2_PUBLIC_IP>:8081/repository/maven-releases/</url>
    </repository>
</distributionManagement>
```
<img width="684" height="418" alt="30" src="https://github.com/user-attachments/assets/ec9a708a-72e8-4d31-85d6-481158ce17eb" />

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
<img width="930" height="463" alt="31" src="https://github.com/user-attachments/assets/1b88147b-ec55-4639-a8a9-ca850e2b3073" />

<img width="956" height="418" alt="33" src="https://github.com/user-attachments/assets/cfbb8646-996f-4841-b926-761ee59543c6" />
<img width="944" height="416" alt="34" src="https://github.com/user-attachments/assets/50b58885-70d2-4423-9aaf-e55390b45618" />
<img width="957" height="395" alt="35" src="https://github.com/user-attachments/assets/120c62cc-ad4b-40d9-8cc5-3a621bbdabf4" />


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
<img width="654" height="40" alt="40" src="https://github.com/user-attachments/assets/720e9077-d6af-45e5-9032-0a5f2d65a3bf" />

<img width="828" height="458" alt="42" src="https://github.com/user-attachments/assets/27940f2a-a322-4e6c-a1c9-08bd15a8d75f" />

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
<img width="955" height="391" alt="36" src="https://github.com/user-attachments/assets/ece1c17b-13ac-4a24-92f1-092a5e4fd6c6" />
<img width="955" height="391" alt="36" src="https://github.com/user-attachments/assets/46ad134b-58b0-48e9-abb7-de05dd115ddd" />
<img width="941" height="340" alt="38" src="https://github.com/user-attachments/assets/6cc38a0f-cebf-41cc-b29b-fd355d8dfac0" />
<img width="937" height="445" alt="39" src="https://github.com/user-attachments/assets/ba387537-6f43-48dc-b75f-4d3c336e9df6" />
<img width="940" height="165" alt="41" src="https://github.com/user-attachments/assets/cd5968f5-d6a2-4653-926f-05fc9937497f" />






