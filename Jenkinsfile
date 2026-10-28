pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t devops-deployment-monitor .'
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker rm -f devops-deployment-monitor-container || true

                    docker run -d \
                      --restart unless-stopped \
                      -p 5004:5000 \
                      --name devops-deployment-monitor-container \
                      devops-deployment-monitor
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:5004/health; then
                            echo "Application is healthy"
                            exit 0
                        fi
                        sleep 2
                    done

                    docker logs devops-deployment-monitor-container
                    exit 1
                '''
            }
        }
    }
}
