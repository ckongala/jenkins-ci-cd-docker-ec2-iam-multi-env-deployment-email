pipeline {
    agent { label 'Jenkins-Agent-1' }

    environment {
        ECR_REPO = 'public.ecr.aws/y8h2p9f1/chinni/jenkins-ecr'
        IMAGE_NAME = 'flaskapp-jenkins'
        TAG = "${env.BRANCH_NAME}-${env.BUILD_ID}"
        PORT = "${env.BRANCH_NAME == 'dev' ? '5001' : (env.BRANCH_NAME == 'staging' ? '5002' : '5003')}"
        CONTAINER_NAME = "${IMAGE_NAME}-${env.BRANCH_NAME}"
        AWS_REGION = 'ap-south-1'  // Ensure region is defined if needed in other steps
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/ckongala/jenkins-ci-cd-docker-ec2-iam-multi-env-deployment-email.git'
                echo "Checkout completed successfully"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t ${env.ECR_REPO}:${env.TAG} ."
                    echo "Docker image build completed successfully"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    // Authenticate to AWS ECR and push the Docker image
                    sh "aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${env.ECR_REPO}"
                    sh "docker push ${env.ECR_REPO}:${env.TAG}"
                    echo "Docker image pushed to ECR successfully"
                }
            }
            post {
                success {
                    // Send email notification after successful image push to ECR
                    emailext(
                        subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
                        body: "Hello,\n\nThe Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.\n\nBest regards,\nJenkins",
                        to: "chinnikrishna2023@gmail.com"
                    )
                    echo "Success email notification sent!"
                }
            }
        }

        stage('Static Code Analysis - SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mvn sonar:sonar'
                        echo "Static code analysis with SonarQube completed successfully"
                    }
                }
            }
        }

        stage('Container Security Scan - Trivy') {
            steps {
                script {
                    // Run container security scan with Trivy
                    sh "trivy image ${env.ECR_REPO}:${env.TAG}"
                    echo "Container security scan completed successfully"
                }
            }
        }

        stage('Cleanup Previous Containers') {
            steps {
                script {
                    // Stop and remove any existing container with the same name
                    sh "docker stop ${env.CONTAINER_NAME} || true"
                    sh "docker rm ${env.CONTAINER_NAME} || true"
                    // Free up the designated port
                    sh "fuser -k ${env.PORT}/tcp || true"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Run the Docker container for testing on the branch-specific port
                    sh "docker run -d --name ${env.CONTAINER_NAME} -p ${env.PORT}:5000 ${env.ECR_REPO}:${env.TAG}"
                    sh 'sleep 5' // Wait for the container to start

                    // Test if the Flask app is responding on the designated port
                    sh "curl -f http://localhost:${env.PORT} || exit 1"

                    // Stop and remove the test container after testing
                    sh "docker stop ${env.CONTAINER_NAME}"
                    sh "docker rm ${env.CONTAINER_NAME}"
                }
            }
        }

        stage('Deploy to EC2') {
            when {
                branch 'main' // Only deploy if on the main branch
            }
            steps {
                script {
                    sh """
                        docker pull ${env.ECR_REPO}:${env.TAG}  
                        docker stop ${env.IMAGE_NAME} || true  
                        docker rm ${env.IMAGE_NAME} || true 
                        docker run -d --name ${env.IMAGE_NAME} -p 8000:5000 ${env.ECR_REPO}:${env.TAG}
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean workspace after the job completes
            cleanWs()
        }

        success {
            // Send email notification after successful build
            emailext(
                subject: "Jenkins Pipeline - Build Success",
                body: "Hello,\n\nThe Jenkins pipeline has completed successfully.\n\nBest regards,\nJenkins",
                to: "chinnikrishna2023@gmail.com"
            )
            echo "Pipeline completed successfully!"
        }

        failure {
            // Send email notification on failure
            emailext(
                subject: "Jenkins Pipeline - Build Failed",
                body: "Hello,\n\nThe Jenkins pipeline has failed. Please check the logs for more details.\n\nBest regards,\nJenkins",
                to: "chinnikrishna2023@gmail.com"
            )
            echo "Pipeline failed!"
        }
    }
}
