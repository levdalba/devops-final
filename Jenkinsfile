pipeline {
    agent any

    stages {
        stage('Setup & Install') {
            steps {
                script {
                    echo 'Installing Node.js 24 and Dependencies...'

                    sh '''
                        curl -fsSL https://deb.nodesource.com/setup_24.x -o nodesource_setup.sh
                        sudo -E bash nodesource_setup.sh
                        sudo apt-get install -y nodejs
                    '''

                    sh 'npm install express'
                }
            }
        }

        stage('Unit Tests') {
            steps {
                script {
                    echo 'Running Unit Tests...'

                    sh 'node --test'
                }
            }
        }

        stage('Deploy to Systemd') {
            steps {
                script {
                    echo 'Deploying to Systemd...'
                    sh '''
                        # 1. Prepare directory
                        sudo mkdir -p /opt/myapp

                        # 2. Copy files (Code + node_modules)
                        sudo cp index.js /opt/myapp/
                        sudo cp -r node_modules /opt/myapp/

                        # 3. Setup Service
                        # Assuming myapp.service is in workspace
                        sudo cp myapp.service /etc/systemd/system/myapp.service

                        # 4. Reload and Start
                        sudo systemctl daemon-reload
                        sudo systemctl enable myapp
                        sudo systemctl restart myapp
                    '''
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                script {
                    echo 'Building and Running Docker Container...'
                    sh '''
                        # Build Image
                        sudo docker build -t myapp:latest .

                        # Cleanup old container if exists
                        sudo docker rm -f myapp-container || true

                        # Run new container
                        sudo docker run -d -p 4444:4444 --name myapp-container myapp:latest
                    '''
                }
            }
        }

        // --- 5. Deploy to Kubernetes (10% Grade) ---
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    sh '''
                        kubectl apply -f k8s-deployment.yaml --validate=false
                    '''
                }
            }
        }
    }
}