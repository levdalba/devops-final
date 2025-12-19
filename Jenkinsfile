pipeline {
    agent any

    environment {
        TARGET_HOST = 'target' 
        DOCKER_HOST = 'docker'
        K8S_HOST = 'kubernetes' 
    }

    stages {
        stage('Install & Test (Local)') {
            steps {
                script {
                    sh 'npm install express'
                    sh 'node --test'
                }
            }
        }

        stage('Deploy to Target (Systemd)') {
            steps {
                script {
                    echo 'Deploying to Target VM...'
                    sh "scp -o StrictHostKeyChecking=no index.js ${TARGET_HOST}:/opt/myapp/"
                    sh "scp -o StrictHostKeyChecking=no -r node_modules ${TARGET_HOST}:/opt/myapp/"
                    sh "scp -o StrictHostKeyChecking=no myapp.service ${TARGET_HOST}:/etc/systemd/system/"

                    sh """
                        ssh -o StrictHostKeyChecking=no ${TARGET_HOST} '
                            systemctl daemon-reload
                            systemctl enable myapp
                            systemctl restart myapp
                        '
                    """
                }
            }
        }

        stage('Deploy to Docker Host') {
            steps {
                script {
                    echo 'Deploying to Docker Host...'

                    sh "ssh -o StrictHostKeyChecking=no ${DOCKER_HOST} 'mkdir -p /tmp/myapp_build'"
                    sh "scp -o StrictHostKeyChecking=no Dockerfile index.js ${DOCKER_HOST}:/tmp/myapp_build/"
                    sh "scp -o StrictHostKeyChecking=no -r node_modules ${DOCKER_HOST}:/tmp/myapp_build/"


                    sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_HOST} '
                            cd /tmp/myapp_build
                            docker build -t myapp:latest .
                            docker stop myapp-container || true
                            docker rm myapp-container || true
                            docker run -d -p 4444:4444 --name myapp-container myapp:latest
                        '
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    
                    sh "ssh -o StrictHostKeyChecking=no ${K8S_HOST} 'mkdir -p /tmp/k8s_build'"
                    sh "scp -o StrictHostKeyChecking=no Dockerfile index.js ${K8S_HOST}:/tmp/k8s_build/"
                    sh "scp -o StrictHostKeyChecking=no -r node_modules ${K8S_HOST}:/tmp/k8s_build/"
                    
                    sh """
                        ssh -o StrictHostKeyChecking=no ${K8S_HOST} '
                            cd /tmp/k8s_build
                            nerdctl build -t myapp:latest . || docker build -t myapp:latest .
                        '
                    """

                    sh "scp -o StrictHostKeyChecking=no k8s-deployment.yaml ${K8S_HOST}:/tmp/"
                    sh "ssh -o StrictHostKeyChecking=no ${K8S_HOST} 'kubectl apply -f /tmp/k8s-deployment.yaml'"
                }
            }
        }
    }
}
