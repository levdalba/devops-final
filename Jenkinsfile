pipeline {
    agent any

    environment {
        SSH_CREDS_ID = 'lab-deploy-key'
    }

    stages {
        stage('Setup Environment') {
            steps {
                script {
                    sh 'curl -fsSL https://deb.nodesource.com/setup_24.x -o nodesource_setup.sh'
                    sh 'sudo -E bash nodesource_setup.sh'
                    sh 'sudo apt-get install -y nodejs ansible'
                    
                    sh 'npm install express'
                }
            }
        }

        stage('Unit Tests') {
            steps {
                script {
                    sh 'node --test'
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sshagent([SSH_CREDS_ID]) {
                    script {
                        sh 'ansible-playbook -i inventory.ini playbook.yaml'
                    }
                }
            }
        }
    }
}
