pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "784065479571"
        REGION = "us-west-1"
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/vote"
        IMAGE_NAME = "vote"  // Change this for other services (worker, result)
        SONAR_HOST_URL = "http://34.229.95.47:9000"
        SONAR_PROJECT_KEY = "vote"
        SONAR_SCANNER_CLI = "/opt/sonar-scanner/bin/sonar-scanner"
    }

    stages {
        stage('Clone Repository') {
            steps {
               git branch: 'main', credentialsId: 'git-voting-token', url: 'https://github.com/Faoziyah/vote.git'  // Update per service
            }
        }

    stage('SonarQube analysis') {
      steps {
        script {
            scannerHome = tool '<SonarQube>'// must match the name of an actual scanner installation directory on your Jenkins build agent
        }
        withSonarQubeEnv('<sonarqubeInstallation>') {// If you have configured more than one global server connection, you can specify its name as configured in Jenkins
          sh "${scannerHome}/bin/sonar-scanner"
        }
      }
    }


       

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
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

