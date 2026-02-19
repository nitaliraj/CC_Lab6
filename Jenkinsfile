pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building backend Docker image..."

                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Cleaning old backend containers..."

                docker network create app-network || true

                docker rm -f backend1 backend2 || true

                # wait until containers are fully removed
                while docker ps -a --format '{{.Names}}' | grep -E 'backend1|backend2'; do
                    echo "Waiting for containers to be removed..."
                    sleep 2
                done

                echo "Starting backend containers..."

                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Deploying NGINX..."

                docker rm -f nginx-lb || true

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx:latest

                sleep 5

                docker cp ./nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf || true

                docker restart nginx-lb
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running on localhost.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
