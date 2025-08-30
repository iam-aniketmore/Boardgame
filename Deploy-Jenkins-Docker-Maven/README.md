# Boardgame Project - Dockerized Application Deployment with CI/CD Pipeline with Jenkins, Docker, and Maven

This project demonstrates the implementation of a Continuous Integration and Continuous Deployment (CI/CD) pipeline using **Jenkins**, **Docker**, and **Maven** for building and deploying a **Boardgame** application.

## Prerequisites

- **Amazon EC2 Instance**
- **Java** installed on the server
- **Docker** installed
- **Maven** installed
- **Jenkins** set up and running
  
## Project Setup

### Step 1: Setup Server on AWS EC2 Instance

1. **Create an Amazon Linux EC2 Instance**
    - Launch a new EC2 instance using the Amazon Linux AMI.
    - Ensure that the instance has enough resources and an open port for Jenkins (default is 8080).
    - Make sure to enable ports 22 (SSH) and 8080 (Jenkins) in the security group for access.

2. **Install Java**
    - Java is required to run Jenkins and Maven. Run the following commands to install Java 17:
    ```bash
    sudo yum install java-17* -y
    ```

3. **Install Jenkins**
    - Download and install Jenkins by following these steps:
    ```bash
    # Add Jenkins repository
    sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    sudo yum upgrade

    # Install dependencies and Jenkins
    sudo yum install fontconfig java-21-openjdk
    sudo yum install jenkins

    # Start Jenkins service
    sudo systemctl daemon-reload
    sudo systemctl enable jenkins
    sudo systemctl start jenkins
    sudo systemctl status jenkins
    ```

4. **Install Docker**
    - Docker is required to build and deploy the application in containers. Run the following command:
    ```bash
    sudo yum install docker -y
    ```

5. **Install Maven**
    - Download and install Maven 3.9.6:
    ```bash
    cd /opt
    sudo wget https://archive.apache.org/dist/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz
    tar -xzvf apache-maven-3.9.6-bin.tar.gz
    mv apache-maven-3.9.6-bin maven
    ```

6. **Allow Jenkins Access**
    - Open port 8080 on your EC2 instance to allow access to Jenkins from a web browser:
    - Access Jenkins at `http://<public-ip>:8080`

7. **Get Jenkins Initial Admin Password**
    - Run the following command to get the Jenkins access password:
    ```bash
    cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
    username `admin` password `admin`
    fullname `admin` email `admin@gmail.com`

9. **Login to Jenkins**
    - Log into Jenkins using the password retrieved above.
    - In Jenkins, go to the **Manage Jenkins** section to install the following plugins:
        - **Docker plugin**
        - **Docker Pipeline**
        - **Maven Integration plugin**

### Step 2: Create Jenkins Pipeline

1. **Configure Tools in Jenkins**
    - In the **Tools** section of Jenkins, add the following tools:
        - Git
        - Maven (set the name to `mvn` and path `/opt/maven`)
        - JDK (set the name to `java-17` and path `/usr/lib/jvm/java-27-amazon-corretto.x86_64`)

2. **Create Jenkins Pipeline Script**
    - Create a new pipeline job in Jenkins and add the following pipeline script and save and apply:
    ```groovy
    pipeline {
        agent any
        tools {
            maven 'mvn'
            jdk 'java-17'
        }

        stages {
            stage('Checkout') {
                steps {
                    git branch: 'main', url: 'https://github.com/iam-aniketmore/Boardgame.git' 
                }
            }
            stage('Compile') {
                steps {
                    sh 'mvn compile'
                }
            }
            stage('Test') {
                steps {
                    sh 'mvn test'
                }
            }
            stage('Package') {
                steps {
                    sh 'mvn package'
                }
            }
            stage('Build Docker Image') {
                steps {
                    sh "docker build -t aniketmore/boardgame:latest ."
                }
            }
            stage('Deploy to Docker') {
                steps {
                    sh """
                    # Stop and remove old container if exists
                    docker stop boardgame || true
                    docker rm boardgame || true

                    # Run new container
                    docker run -dt --name boardgame -p 9090:8080 aniketmore/boardgame:latest
                    """
                }
            }
        }
    }
    ```

3. **Run the Pipeline**
    - Trigger the pipeline to run the build, tests, and deployment stages.
    - Once the pipeline completes, check if the application is deployed successfully by navigating to `http://<public-ip>:9090`.

### Step 3: Access the Application

After successfully running the Jenkins pipeline, your **Boardgame** application should be accessible in your browser at the following URL: 
`http://<public-ip>:9090`

This confirms that the application has been deployed inside a Docker container running on your EC2 instance.

## Project Details

- **Jenkins** is used for automating the build, test, and deployment pipeline.
- **Docker** is used to containerize the application and deploy it in a consistent and scalable environment.
- **Maven** is used for building and managing the project dependencies.
- The application is a Boardgame simulation that showcases continuous integration and continuous deployment.

## Prerequisites

- **Amazon EC2 Instance**
- **Java** installed on the server
- **Docker** installed
- **Maven** installed
- **Jenkins** set up and running

## Troubleshooting

- **Jenkins not starting**: Ensure Java is installed properly and there are no errors in the Jenkins logs.
- **Docker image build failed**: Check the Dockerfile for issues or missing dependencies.
- **Port 8080 not open**: Make sure the EC2 security group allows inbound traffic on port 8080.

