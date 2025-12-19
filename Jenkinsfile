pipeline {
    agent any

    environment {
        SSH_CREDS_ID = 'lab-deploy-key'
    }

    stages {
        stage('Setup Environment') {
            steps {
                script {
                    echo 'Installing Dependencies...'
                    sh 'sudo apt-get update'
                    sh 'sudo apt-get install -y ansible nodejs'
                    
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
