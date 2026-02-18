pipeline {
    agent any
    environment {
        NETWORK_NAME = "jenkins-net"
    }
    stages {
        stage('Build Backend') {
            steps {
                script {
                    // Create network if not exists
                    sh 'docker network create $NETWORK_NAME || true'
                    // Build backend image
                    sh 'docker build -t backend-app backend'
                }
            }
        }
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                    # Remove existing containers
                    docker rm -f backend1 backend2 || true
                    
                    # Run backend containers on the custom network
                    docker run -d --name backend1 --network $NETWORK_NAME backend-app
                    docker run -d --name backend2 --network $NETWORK_NAME backend-app
                    
                    # Wait for backends to initialize
                    sleep 3
                '''
            }
        }
        stage('Deploy Nginx Load Balancer') {
            steps {
                sh '''
                    # Remove existing nginx container
                    docker rm -f nginx-lb || true
                    
                    # Run Nginx on the same network, exposing port 80
                    docker run -d -p 80:80 --name nginx-lb --network $NETWORK_NAME nginx
                    
                    # Wait for Nginx to start
                    sleep 2
                    
                    # Copy custom configuration
                    docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                    
                    # Reload Nginx configuration
                    docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
}
