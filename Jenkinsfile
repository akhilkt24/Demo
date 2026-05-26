pipeline {
    agent { label 'docker' }

    environment {
        IMAGE_NAME     = "akhilkt24/demo-website"
        CONTAINER_NAME = "demo-site"
        APP_PORT       = "80"
        EMAIL_TO       = "iamakhilkt@gmail.com"
        APP_NAME       = "Demo Website"
        TEAM_NAME      = "Akhil DevOps Team"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Pulling code from GitHub..."
                git branch: 'main', url: 'https://github.com/akhilkt24/Demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh '''
                docker build -t $IMAGE_NAME:$BUILD_NUMBER .
                docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest
                echo "Image built successfully!"
                docker images | grep demo-website
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing image to Docker Hub..."
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$BUILD_NUMBER
                    docker push $IMAGE_NAME:latest
                    echo "Successfully pushed to Docker Hub!"
                    '''
                }
            }
        }

        stage('Manual Approval') {
            steps {
                input message: "Approve deployment to production?", ok: "Deploy"
            }
        }

        stage('Deploy Container') {
            steps {
                echo "Deploying container..."
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                docker pull $IMAGE_NAME:latest
                docker run -d \
                    --name $CONTAINER_NAME \
                    --restart always \
                    -p $APP_PORT:80 \
                    $IMAGE_NAME:latest
                echo "Container deployed!"
                docker ps | grep $CONTAINER_NAME
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo "Running health check..."
                sh '''
                sleep 5
                STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:80)
                echo "HTTP Status: $STATUS"
                if [ "$STATUS" = "200" ]; then
                    echo "Health check PASSED!"
                else
                    echo "Health check FAILED!"
                    exit 1
                fi
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
            mail to: "${EMAIL_TO}",
            mimeType: 'text/html',
            subject: "SUCCESS [${APP_NAME}] Build #${BUILD_NUMBER} Deployed!",
            body: """<html><body style='font-family:Arial,sans-serif;background:#f4f4f4;'><div style='max-width:600px;margin:30px auto;background:#fff;border-radius:8px;overflow:hidden;'><div style='background:#1a7f4b;padding:24px 32px;'><h1 style='color:#fff;margin:0;'>Deployment Successful</h1><p style='color:#a8f0c6;margin:6px 0 0;'>${APP_NAME} is live!</p></div><div style='background:#d4edda;padding:16px 32px;border-left:4px solid #1a7f4b;'><p style='margin:0;color:#155724;font-weight:bold;'>BUILD PASSED - All stages completed successfully</p></div><div style='padding:24px 32px;'><table style='width:100%;font-size:14px;'><tr><td style='color:#666;width:40%;padding:8px 0;'>Application</td><td>${APP_NAME}</td></tr><tr style='background:#f9f9f9;'><td style='color:#666;padding:8px 0;'>Build Number</td><td>#${BUILD_NUMBER}</td></tr><tr><td style='color:#666;padding:8px 0;'>Docker Image</td><td>${IMAGE_NAME}:${BUILD_NUMBER}</td></tr><tr style='background:#f9f9f9;'><td style='color:#666;padding:8px 0;'>Docker Hub</td><td>hub.docker.com/r/akhilkt24/demo-website</td></tr><tr><td style='color:#666;padding:8px 0;'>Team</td><td>${TEAM_NAME}</td></tr></table></div><div style='padding:0 32px 24px;'><a href='${BUILD_URL}' style='background:#1a7f4b;color:#fff;padding:12px 24px;text-decoration:none;border-radius:4px;'>View Build Details</a></div><div style='background:#f4f4f4;padding:16px;text-align:center;'><p style='color:#999;font-size:12px;margin:0;'>Jenkins CI/CD | ${TEAM_NAME}</p></div></div></body></html>"""
        }
        failure {
            mail to: "${EMAIL_TO}",
            mimeType: 'text/html',
            subject: "FAILED [${APP_NAME}] Build #${BUILD_NUMBER} - Action Required",
            body: """<html><body style='font-family:Arial,sans-serif;background:#f4f4f4;'><div style='max-width:600px;margin:30px auto;background:#fff;border-radius:8px;overflow:hidden;'><div style='background:#c0392b;padding:24px 32px;'><h1 style='color:#fff;margin:0;'>Deployment Failed</h1><p style='color:#f5b7b1;margin:6px 0 0;'>${APP_NAME} - action required</p></div><div style='background:#f8d7da;padding:16px 32px;border-left:4px solid #c0392b;'><p style='margin:0;color:#721c24;font-weight:bold;'>BUILD FAILED</p></div><div style='padding:24px 32px;'><table style='width:100%;font-size:14px;'><tr><td style='color:#666;width:40%;padding:8px 0;'>Application</td><td>${APP_NAME}</td></tr><tr style='background:#f9f9f9;'><td style='color:#666;padding:8px 0;'>Build Number</td><td style='color:#c0392b;'>#${BUILD_NUMBER}</td></tr><tr><td style='color:#666;padding:8px 0;'>Team</td><td>${TEAM_NAME}</td></tr></table></div><div style='padding:0 32px 24px;'><ol style='font-size:14px;line-height:1.8;'><li>Check console output</li><li>Fix the issue</li><li>Push fix to GitHub</li><li>Jenkins will rebuild automatically</li></ol><a href='${BUILD_URL}console' style='background:#c0392b;color:#fff;padding:12px 24px;text-decoration:none;border-radius:4px;'>View Console</a></div><div style='background:#f4f4f4;padding:16px;text-align:center;'><p style='color:#999;font-size:12px;margin:0;'>Jenkins CI/CD | ${TEAM_NAME}</p></div></div></body></html>"""
        }
        always {
            sh 'docker logout || true'
            echo "Pipeline finished."
        }
    }
}
