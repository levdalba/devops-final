pipeline {
    agent any

    environment {
        TARGET_HOST = 'target' 
        DOCKER_HOST = 'docker'
        K8S_HOST = 'kubernetes'
        SSH_CREDS_ID = 'lab-deploy-key'
    }

    stages {
        stage('Setup Jenkins Agent') {
            steps {
                script {
                    def nodeExists = sh(script: 'node -v', returnStatus: true) == 0
                    if (!nodeExists) {
                        echo 'Installing Node.js...'
                        sh 'curl -fsSL https://deb.nodesource.com/setup_24.x -o nodesource_setup.sh'
                        sh 'sudo -E bash nodesource_setup.sh'
                        sh 'sudo apt-get install -y nodejs'
                    }
                }
            }
        }

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
                sshagent([SSH_CREDS_ID]) {
                    script {
                        echo 'Deploying to Target...'
                        
                        sh "ssh -o StrictHostKeyChecking=no root@${TARGET_HOST} 'mkdir -p /opt/myapp'"
                        sh "scp -o StrictHostKeyChecking=no index.js root@${TARGET_HOST}:/tmp/index.js"
                        sh "scp -o StrictHostKeyChecking=no -r node_modules root@${TARGET_HOST}:/tmp/node_modules"
                        sh "scp -o StrictHostKeyChecking=no myapp.service root@${TARGET_HOST}:/tmp/myapp.service"
                        
                        sh """
                            ssh -o StrictHostKeyChecking=no root@${TARGET_HOST} '
                                mv /tmp/index.js /opt/myapp/
                                rm -rf /opt/myapp/node_modules && mv /tmp/node_modules /opt/myapp/
                                mv /tmp/myapp.service /etc/systemd/system/
                                systemctl daemon-reload
                                systemctl enable myapp
                                systemctl restart myapp
                            '
                        """
                    }
                }
            }
        }

        stage('Deploy to Docker Host') {
            steps {
                sshagent([SSH_CREDS_ID]) {
                    script {
                        echo 'Deploying to Docker...'
                        sh "ssh -o StrictHostKeyChecking=no root@${DOCKER_HOST} 'mkdir -p /tmp/myapp_build'"
                        sh "scp -o StrictHostKeyChecking=no Dockerfile index.js root@${DOCKER_HOST}:/tmp/myapp_build/"
                        sh "scp -o StrictHostKeyChecking=no -r node_modules root@${DOCKER_HOST}:/tmp/myapp_build/"

                        sh """
                            ssh -o StrictHostKeyChecking=no root@${DOCKER_HOST} '
                                cd /tmp/myapp_build
                                docker build -t myapp:latest .
                                docker rm -f myapp-container || true
                                docker run -d -p 4444:4444 --name myapp-container myapp:latest
                            '
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent([SSH_CREDS_ID]) {
                    script {
                        echo 'Deploying to Kubernetes...'
                        sh "ssh -o StrictHostKeyChecking=no root@${K8S_HOST} 'mkdir -p /tmp/k8s_build'"
                        sh "scp -o StrictHostKeyChecking=no Dockerfile index.js root@${K8S_HOST}:/tmp/k8s_build/"
                        sh "scp -o StrictHostKeyChecking=no -r node_modules root@${K8S_HOST}:/tmp/k8s_build/"
                        
                        sh """
                            ssh -o StrictHostKeyChecking=no root@${K8S_HOST} '
                                cd /tmp/k8s_build
                                if command -v nerdctl &> /dev/null; then
                                    nerdctl build -n k8s.io -t myapp:latest .
                                else
                                    docker build -t myapp:latest .
                                fi
                            '
                        """

                        sh "scp -o StrictHostKeyChecking=no k8s-deployment.yaml root@${K8S_HOST}:/tmp/"
                        sh "ssh -o StrictHostKeyChecking=no root@${K8S_HOST} 'kubectl apply -f /tmp/k8s-deployment.yaml'"
                    }
                }
            }
        }
    }
}
