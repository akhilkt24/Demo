pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/akhilkt24/Demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t frozenyogurt-app .'
            }
        }

        stage('Stop Old Container') {
            steps {
                bat 'docker rm -f frozenyogurt || exit 0'
            }
        }

        stage('Run Container') {
            steps {
                bat 'docker run -d -p 8080:80 --name frozenyogurt frozenyogurt-app'
            }
        }
    }
}
