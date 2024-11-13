pipeline {
    agent { label 'Jenkins-Agent-1' }

    environment {
        ECR_REPO = 'public.ecr.aws/y8h2p9f1/chinni/jenkins-ecr'
        IMAGE_NAME = 'flaskapp-jenkins'
        TAG = "${env.BRANCH_NAME}-${env.BUILD_ID}"
        SSH_KEY = credentials('ubuntu')  // Assuming this is your SSH key for EC2 access
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the GitHub repository containing the Dockerfile, app.py, and Jenkinsfile
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/ckongala/jenkins-ci-cd-docker-ec2-iam-multi-env-deployment-email.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image from the Dockerfile in the repo
                    sh """
                    docker build -t ${env.IMAGE_NAME}:${env.TAG} .
                    """
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    // Use AWS CLI to login to ECR and push the Docker image
                    sh """
                    $(aws ecr get-login-password --region ap-south-1) | docker login --username AWS --password-stdin ${env.ECR_REPO}
                    docker tag ${env.IMAGE_NAME}:${env.TAG} ${env.ECR_REPO}:${env.TAG}
                    docker push ${env.ECR_REPO}:${env.TAG}
                    """
                }
            }
            post {
                success {
                    // Send email notification after successful image push to ECR
                    emailext(
                        subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
                        body: "Hello,\n\nThe Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.\n\nBest regards,\nJenkins",
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                        to: "chinnikrishna2023@gmail.com"
                    )
                }
            }
        }

        stage('Static Code Analysis - SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Container Security Scan - Trivy') {
            steps {
                script {
                    // Perform a security scan on the Docker image using Trivy
                    sh "trivy image ${env.ECR_REPO}:${env.TAG}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    def targetHost = '3.110.171.221'  // EC2 instance IP

                    // Using the SSH key to connect to the EC2 instance and deploy the container
                    sshagent([SSH_KEY]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ubuntu@${targetHost} << EOF
                        docker pull ${env.ECR_REPO}:${env.TAG}  // Pull the Docker image from ECR
                        docker stop ${env.IMAGE_NAME} || true  // Stop any running container with the same name
                        docker rm ${env.IMAGE_NAME} || true  // Remove the old container
                        docker run -d --name ${env.IMAGE_NAME} -p 8000:5000 ${env.ECR_REPO}:${env.TAG}  // Run the new container
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // Clean up the workspace after the build
        }
    }
}
