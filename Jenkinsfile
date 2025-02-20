pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "784065479571"
        REGION = "us-west-1"
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/vote"
        IMAGE_NAME = "vote"  // Change this for other services (worker, result)
        SONAR_HOST_URL = "http://34.229.95.47:9000/"
        SONAR_PROJECT_KEY = "sonar-token"
        SONAR_SCANNER_CLI = "/opt/sonar-scanner/bin/sonar-scanner"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Faoziyah/vote.git'  // Update per service
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {  // Ensure 'SonarQube' is configured in Jenkins
                    sh """
                    $SONAR_SCANNER_CLI -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                                       -Dsonar.sources=. \
                                       -Dsonar.host.url=$SONAR_HOST_URL
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_REPO
                docker tag ${IMAGE_NAME}:latest $ECR_REPO:latest
                docker push $ECR_REPO:latest
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline execution successful!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}

