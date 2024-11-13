pipeline {
    agent { label 'Jenkins-Agent-1' }

    environment {
        ECR_REPO = 'public.ecr.aws/y8h2p9f1/chinni/jenkins-ecr'
        IMAGE_NAME = 'flaskapp-jenkins'
        TAG = "${env.BRANCH_NAME}-${env.BUILD_ID}"
    }

    stages {
        
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/ckongala/jenkins-ci-cd-docker-ec2-iam-multi-env-deployment-email.git'
                echo "1st stage checkout is success"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh """
                    docker build -t ${env.IMAGE_NAME}:${env.TAG} .
                    """
                    echo "2nd stage build docker image is success"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    // Use IAM role-based authentication to interact with ECR
                    sh """
                    LOGIN_COMMAND=\$(aws ecr get-login-password --region ap-south-1)
                    echo \$LOGIN_COMMAND | docker login --username AWS --password-stdin ${env.ECR_REPO}
                    docker tag ${env.IMAGE_NAME}:${env.TAG} ${env.ECR_REPO}:${env.TAG}
                    docker push ${env.ECR_REPO}:${env.TAG}
                    """
                    echo "3rd stage push to ECR is success"
                }
            }
            post {
                success {
                    // Send email notification after successful image push to ECR
                    // emailext(
                    //     subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
                    //     body: "Hello,\n\nThe Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.\n\nBest regards,\nJenkins",
                    //     to: "chinnikrishna2023@gmail.com"
                    // )
                    echo "Post success notification sent!"
                }
            }
        }

        stage('Static Code Analysis - SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mvn sonar:sonar'
                        echo "Static Code Analysis - SonarQube is success"
                    }
                }
            }
        }

        stage('Container Security Scan - Trivy') {
            steps {
                script {
                    // Container security scan with Trivy
                    sh "trivy image ${env.ECR_REPO}:${env.TAG}"
                    echo "Container Security Scan - Trivy is success"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    // Deploy to EC2 instance
                    sh """
                    docker pull ${env.ECR_REPO}:${env.TAG}  // Pull the Docker image from ECR
                    docker stop ${env.IMAGE_NAME} || true  // Stop any running container with the same name
                    docker rm ${env.IMAGE_NAME} || true  // Remove the old container
                    docker run -d --name ${env.IMAGE_NAME} -p 8000:5000 ${env.ECR_REPO}:${env.TAG}  // Run the new container
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
            // Send email notification after successful build (if not already sent)
            // emailext(
            //     subject: "Jenkins Pipeline - Build Success",
            //     body: "Hello,\n\nThe Jenkins pipeline has completed successfully.\n\nBest regards,\nJenkins",
            //     to: "chinnikrishna2023@gmail.com"
            // )
            echo "Pipeline completed successfully!"
        }

        failure {
            // Send email notification on failure
            // emailext(
            //     subject: "Jenkins Pipeline - Build Failed",
            //     body: "Hello,\n\nThe Jenkins pipeline has failed. Please check the logs for more details.\n\nBest regards,\nJenkins",
            //     to: "chinnikrishna2023@gmail.com"
            // )
            echo "Pipeline failed!"
        }
    }
}
