pipeline {
    agent { label 'Jenkins-Agent-1' }

    environment {
        ECR_REPO = 'public.ecr.aws/y8h2p9f1/chinni/jenkins-ecr'
        IMAGE_NAME = 'flaskapp-jenkins'
        TAG = "${env.BRANCH_NAME}-${env.BUILD_ID}"
        SSH_KEY = credentials('ubuntu')  // Name of the SSH key stored in Jenkins credentials
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Use 'checkout scm' to check out the code from GitHub
                    checkout scm
                }
            }
        }

        stage('Pull Docker Image from Docker Hub') {
            steps {
                script {
                    // Pull the Docker image from Docker Hub
                    sh "docker pull chinni81/flaskapp:1.0"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    // Login to ECR and push the image
                    sh """
                    LOGIN_COMMAND=\$(aws ecr get-login-password --region ap-south-1)
                    echo \$LOGIN_COMMAND | docker login --username AWS --password-stdin ${env.ECR_REPO}
                    docker tag chinni81/flaskapp:1.0 ${env.ECR_REPO}:${env.TAG}
                    docker push ${env.ECR_REPO}:${env.TAG}
                    """
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
                    // Run Trivy to scan the Docker image for vulnerabilities
                    sh "trivy image ${ECR_REPO}:${TAG}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    def targetHost = '3.110.171.221'  // Replace with the actual EC2 instance IP

                    // Use SSH key to connect to the EC2 instance and deploy
                    sshagent([SSH_KEY]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${targetHost} << EOF
                        docker pull ${ECR_REPO}:${TAG}
                        docker stop ${IMAGE_NAME} || true
                        docker rm ${IMAGE_NAME} || true
                        docker run -d --name ${IMAGE_NAME} -p 8000:5000 ${ECR_REPO}:${TAG}
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            // Send email notification after the successful completion of the job
            emailext(
                subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
                body: "Hello,\n\nThe Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.\n\nBest regards,\nJenkins",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "chinnikrishna2023@gmail.com"
            )
        }
        
        always {
            // Clean up workspace after the build
            cleanWs()  // This will clean the workspace regardless of the result
        }
        
        failure {
            // Optional: Send a failure email if the build fails
            emailext(
                subject: "Jenkins Job Failed",
                body: "Hello,\n\nThe Jenkins job has failed during the pipeline execution.\n\nPlease check the Jenkins logs for more details.\n\nBest regards,\nJenkins",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "chinnikrishna2023@gmail.com"
            )
        }
    }
}
