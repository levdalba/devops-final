pipeline {
    agent any

    environment {
        TARGET_HOST = 'target' 
        DOCKER_HOST = 'docker'
        K8S_HOST = 'kubernetes' 
    }

    stages {
        stage('Setup Jenkins Agent') {
            steps {
                script {
                    def nodeExists = sh(script: 'node -v', returnStatus: true) == 0
                    if (!nodeExists) {
                        echo 'Node.js not found. Installing...'
                        sh '''
                            curl -fsSL https://deb.nodesource.com/setup_24.x -o nodesource_setup.sh
                            sudo -E bash nodesource_setup.sh
                            sudo apt-get install -y nodejs
                        '''
                    } else {
                        echo 'Node.js is already installed.'
                    }
                }
            }
        }


        stage('Install & Test (Local)') {
            steps {
                script {
                    echo 'Running Unit Tests...'
                    sh 'npm install express'
                    sh 'node --test'
                }
            }
        }


        stage('Deploy to Target (Systemd)') {
            steps {
                script {
                    echo 'Deploying to Target VM...'

                    sh "ssh -o StrictHostKeyChecking=no ${TARGET_HOST} 'sudo mkdir -p /opt/myapp'"
                    

                    sh "scp -o StrictHostKeyChecking=no index.js ${TARGET_HOST}:/tmp/index.js"
                    sh "scp -o StrictHostKeyChecking=no -r node_modules ${TARGET_HOST}:/tmp/node_modules"
                    sh "scp -o StrictHostKeyChecking=no myapp.service ${TARGET_HOST}:/tmp/myapp.service"

                    sh """
                        ssh -o StrictHostKeyChecking=no ${TARGET_HOST} '
                            sudo mv /tmp/index.js /opt/myapp/
                            sudo rm -rf /opt/myapp/node_modules && sudo mv /tmp/node_modules /opt/myapp/
                            sudo mv /tmp/myapp.service /etc/systemd/system/
                            sudo systemctl daemon-reload
                            sudo systemctl enable myapp
                            sudo systemctl restart myapp
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
                            docker rm -f myapp-container || true
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
                            # Try nerdctl first (common in some labs), fallback to docker
                            if command -v nerdctl &> /dev/null; then
                                sudo nerdctl build -n k8s.io -t myapp:latest .
                            else
                                sudo docker build -t myapp:latest .
                            fi
                        '
                    """

                    sh "scp -o StrictHostKeyChecking=no k8s-deployment.yaml ${K8S_HOST}:/tmp/"
                    sh "ssh -o StrictHostKeyChecking=no ${K8S_HOST} 'kubectl apply -f /tmp/k8s-deployment.yaml'"
                }
            }
        }
    }
}
