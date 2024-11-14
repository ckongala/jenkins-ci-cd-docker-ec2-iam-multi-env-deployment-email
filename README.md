# jenkins-ci-cd-docker-ec2-iam-multi-env-deployment-email
Below is an enhanced `README.md` file that fully describes your CI/CD pipeline project, including setup details and a detailed explanation of the Jenkinsfile used to automate the building, testing, and deployment of a Python Flask application.

```markdown
# CI/CD Pipeline with Jenkins, Docker, and AWS

This project demonstrates a Continuous Integration and Continuous Deployment (CI/CD) pipeline using Jenkins with master-slave architecture, Docker containerization, and AWS services. The pipeline automates the deployment of a Python Flask application across multiple environments.

## Technologies Used

- **CI/CD:** Jenkins (master in Docker on an EC2 instance, slave on a separate EC2 instance)
- **Containerization:** Docker
- **Source Control:** GitHub (with branch protection and pull request-based merging)
- **Container Registry:** Amazon ECR
- **Notifications:** Email (using Jenkins Email Extension Plugin)

## Project Structure

```plaintext
/project-directory
│
├── app.py              # Flask application entry point
├── requirements.txt    # Python dependencies
├── Dockerfile          # Docker container specification
└── Jenkinsfile         # Defines the Jenkins pipeline
```

## Setup and Configuration

### GitHub Repository Setup

1. Created a GitHub repository and pushed all files (app.py, requirements.txt, Jenkinsfile, Dockerfile).
2. Configured three branches for environment-specific deployments: `dev`, `staging`, `main`.
3. Applied branch protection rules via GitHub:
   - Settings > Branches > Add rule for `dev`, `staging`, and `main`.
   - Enabled "Require pull request reviews before merging".
   - Enabled "Require status checks to pass before merging".

### Jenkins Setup

#### Jenkins Master Configuration

1. Launched an EC2 instance to serve as the Jenkins Master.
2. Installed Docker, Java, and other necessary tools:
   ```bash
   sudo yum update -y
   sudo yum install docker java-11-openjdk-devel -y
   sudo systemctl start docker
   sudo usermod -aG docker ec2-user
   ```

3. Run Jenkins Master inside a Docker container:
   ```bash
   docker pull jenkins/jenkins:lts
   docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-master \
   -v jenkins_home:/var/jenkins_home \
   -v /var/run/docker.sock:/var/run/docker.sock \
   jenkins/jenkins:lts
   ```

## Accessing Jenkins

Access Jenkins via `http://<Jenkins-Master-EC2-Public-IP>:8080` and complete the initial setup using the secret from `docker exec -it jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword`.

#### Jenkins Slave Configuration

1. Launched another EC2 instance for the Jenkins Slave.
2. Installed Docker and Java:
   ```bash
   sudo yum install docker java-11-openjdk-devel -y
   sudo systemctl start docker
   sudo usermod -aG docker ec2-user
   ```

3. Configured SSH keys for secure communication between Jenkins Master and Slave,
   Generate an SSH key pair on the Jenkins master instance (within the container) if you don’t already have one:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "jenkins-agent-key"
   ssh-copy-id -i ~/.ssh/jenkins-agent-key.pub ec2-user@<Agent-EC2-Public-IP>
   ```
  Add Jenkins Agent in Jenkins UI:
  In Jenkins, go to Manage Jenkins > Manage Nodes and Clouds > New Node.
  Configure the agent node with:
  Remote root directory: /home/ec2-user
  Launch method: SSH
  Host: Agent EC2 public IP
  Credentials: Add SSH private key credentials

5. Added the Jenkins Slave in the Jenkins UI:
   - Manage Jenkins > Manage Nodes and Clouds > New Node.
   - Configured with SSH launch method and provided SSH private key credentials.
      ```
     i install docker and java and all other necesry tools for trobleshooting, 
     in master i created a jenkins container that is runing on 8080 port of aster, 
     so i docker exec -it into container and creatd a SSH key gen 
     i copied that public key into the slave and then try to ssh from inside docker to slave it's working fine, 
     so now i set-up the all the stuff, related to jenkins and now i cretaed a created a node inside the jeniks, to SSH my jenkins into my slave ec2 instance,
     below how i configure 
     name > remote root directory /home/ubuntu/ or /home/ec2-user/ usage :use this as much as possible >
     lauch method, laucnh aget via SSH, HOST ad Slave Ip address, 
     credientials( add copy the rsa (private key generated form jenkins container from master ec2 instance)
     remainig as default.
     ```

### IAM Role Configuration

Created an IAM role (`Jenkins-EC2-Role`) with:
- **AmazonEC2ContainerRegistryFullAccess**
- **SecretsManagerReadWrite**
- Attached to both Jenkins EC2 instances for proper AWS resources access.

### SMTP Settings for Email Notifications

Configured Jenkins for email notifications using the Email Extension Plugin:
- Configured SMTP server settings in Jenkins to use `smtp.gmail.com` with TLS and port 587.
   ```
   first make sure that your gmail has two step authenticaion and, create a new  app password, with random name, and take the password only show only once,
   Jenkins Dashboard > Manage Jenkins > Configure System. 
   setup of SMTp;
   server name as smtp.gamil.com > uses user and password, user as email_id; and password as generated password from app password, 
   uses TLS and port as 587, and verify test, it's working fine.
   Save Settings.
   ```

## Jenkins Pipeline (Jenkinsfile)

The `Jenkinsfile` contains a multi-branch pipeline script that automates:
- **Checkout:** Pulls the latest code from the branch triggered.
- **Build:** Builds a Docker image from the Dockerfile.
- **Push:** Pushes the built image to Amazon ECR.
- **Test:** Deploys the Docker container and runs tests.
- **Deploy:** Deploys the application on production if the `main` branch is triggered.
   ```
   Step 4.3: Create Multi-Branch Pipeline in Jenkins
   Create Multi-Branch Pipeline Job:
   Go to Jenkins Dashboard > New Item and select Multi-branch Pipeline.
   Name it MyApp-MultiBranch-Pipeline and click OK.
   Configure GitHub Repository and Branches:
   In Branch Sources, add GitHub and specify the repository URL.
   Configure branch patterns to detect dev, staging, and main.
   Specify Jenkinsfile Location:
   Under Build Configuration, set it to by Jenkinsfile.
   ```

```groovy
pipeline {
    agent { label 'Jenkins-Agent-1' }
    environment {
        // Environment variables setup
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/yourusername/yourprojectname.git'
            }
        }
        // Additional stages for Build, Push, Test, and Deploy
    }
    post {
        always {
            cleanWs() // Clean workspace after the job is done
        }
        success {
            mail(to: 'your-email@example.com', subject: 'Build Success', body: 'Successful Build')
        }
        failure {
            mail(to: 'your-email@example.com', subject: 'Build Failed', body: 'Failed Build')
        }
    }
}
```
## Conclusion

This project sets a robust foundation for a scalable CI/CD pipeline capable of handling multi-environment deployments with automated testing and notifications, ensuring consistent and reliable delivery cycles.
