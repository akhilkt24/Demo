pipeline {
    agent any

    environment {
        DOCKER_SERVER = "192.168.71.136"
        DOCKER_USER   = "ubuntu"
        APP_NAME      = "Demo-site"
        IMAGE_NAME    = "demo-website"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/akhilkt24/Demo.git'
            }
        }

        stage('Build Docker Image (Local Jenkins)') {
            steps {
                bat 'docker build -t demo-website .'
            }
        }

        stage('Manual Approval') {
            steps {
                input message: "Approve Deployment to Docker Server?", ok: "Deploy"
            }
        }

        stage('Deploy to Docker Server') {
            steps {
                bat """
                ssh ubuntu@${DOCKER_SERVER} ^
                "docker rm -f ${APP_NAME} || true && ^
                 docker run -d -p 80:80 --name ${APP_NAME} ${IMAGE_NAME}"
                """
            }
        }
    }

    post {
        success {
            echo "Deployment Successful"
        }

        failure {
            echo "Deployment Failed"
        }
    }
}
