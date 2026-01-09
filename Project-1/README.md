**Deploy Java Web Application on Docker Using Jenkins on AWS**

  This project demonstrates deploying a Java web application on a Docker container hosted on an AWS EC2 instance, fully automated using Jenkins CI/CD pipeline       integrated with GitHub.

1. Setup Jenkins Server on AWS EC2
2. Setup & Configure Maven and Git
3. Integrate GitHub and Maven with Jenkins
4. Setup Docker Host
5. Integrate Docker with Jenkins
6. Automate Build and Deployment using Jenkins
7. Test the Deployment

Prerequisites:
- AWS Account
- Git/GitHub Account with source code
- Local machine with CLI access
- Familiarity with Docker and Git


 Step 1: Setup Jenkins Server on AWS EC2

1. Launch a Linux EC2 instance (t2.micro recommended for free tier).
2. Select existing key pair or create a new one.
3. Configure Security Group:
   - Allow SSH (port 22)
   - Allow HTTP/Jenkins (port 8080)
4. Connect via SSH:

bash
ssh -i new3112.pem ec2-user@<public-ip>

5. Install Jenkins:
  sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
  sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
  sudo yum install epel-release -y
  sudo yum install java-11-openjdk-devel -y
  sudo yum install jenkins -y
  sudo systemctl enable jenkins
  sudo systemctl start jenkins
6. Access Jenkins Web UI: http://<EC2_PUBLIC_IP>:8080
7. Unlock Jenkins using:
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Step 2: Integrate GitHub with Jenkins
1. Install Git:
   sudo yum install git -y
   git --version
2. Install GitHub plugin in Jenkins (Manage Jenkins → Manage Plugins → Available → GitHub Integration → Install without restart)
3. Configure Git in Jenkins (Manage Jenkins → Global Tool Configuration):
    Name: Git
    Path: /usr/bin/git
4. Save changes.

Step 3: Integrate Maven with Jenkins
1. Download and install Maven:
   cd /opt
   wget https://dlcdn.apache.org/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz
   tar -xzf apache-maven-3.9.5-bin.tar.gz
2. Set environment variables in : sudo nano /etc/profile.d/maven.sh
  export JAVA_HOME=/usr/lib/jvm/default-java
  export M2_HOME=/opt/maven
  export MAVEN_HOME=/opt/maven
  export PATH=${M2_HOME}/bin:${PATH}
3.Install Maven Integration Plugin in Jenkins
4.Configure Java and Maven paths in Jenkins (Manage Jenkins → Global Tool Configuration)

Step 4: Setup Docker Host
1.Launch another EC2 instance for Docker (or use same if preferred)
2.Install Docker:
  sudo yum install docker -y
  sudo systemctl enable docker
  sudo systemctl start docker
  docker --version
3.Run basic Tomcat container:
  docker run -d --name tomcat-container -p 8081:8080 tomcat
4.Copy content from webapps.dist to webapps:
  docker exec -it tomcat-container /bin/bash
  cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
  exit
5.Create custom Dockerfile:
  FROM tomcat:latest
  RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps

Step 5: Integrate Docker with Jenkins
1.Create dockeradmin user and add to docker group:
  sudo useradd dockeradmin
  sudo passwd dockeradmin
  sudo usermod -aG docker dockeradmin
2.Enable password authentication in /etc/ssh/sshd_config:
  PasswordAuthentication yes
3.Reload SSH service:
  sudo systemctl reload sshd
4.Install "Publish Over SSH" plugin in Jenkins (Manage Jenkins → Manage Plugins)
5.Add Docker host SSH credentials and configure Publish Over SSH.

Step 6: Jenkins Job to Build & Deploy to Docker
1.Create Jenkins job → configure to Poll SCM
2.Use Send files or execute commands over SSH post-build step
3.Example commands to build and run Docker container:
  cd /opt/docker
  docker build -t regapp:v1 .
  docker run -d --name registerapp -p 8087:8080 regapp:v1
4.Trigger job automatically on GitHub commits.

Step 7: Update Dockerfile to Include WAR
  FROM tomcat:latest
  RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
  COPY ./*.war /usr/local/tomcat/webapps
Build and run new image:
  docker build -t tomcat:v1 .
  docker run -d --name tomcatv1 -p 8086:8080 tomcat:v1

Access application: http://<DOCKERHOST_PUBLIC_IP>:8086/webapp/

Step 8: Automate End-to-End Deployment
Commit changes to GitHub → triggers Jenkins build → copies WAR to Docker host → builds Docker image → runs container
Example Exec command in Jenkins post-build:
  cd /opt/docker
  docker build -t regapp:v1 .
  docker run -d --name registerapp -p 8087:8080 regapp:v1

Access deployed app: http://<DOCKERHOST_PUBLIC_IP>:8087/webapp/

Conclusion

This project automates the full CI/CD pipeline for a Java web application using GitHub, Jenkins, Docker, and AWS EC2.

