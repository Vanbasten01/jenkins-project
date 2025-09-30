pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }

    stages {
        stage('Set Up') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh "pytest"
            }
        }

        stage('Package code') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git*'"
                sh "ls -lart"
            }
        }

        stage('Deploy to prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'USERNAME')]) {
                    sh """
                        scp -i \$MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip \${USERNAME}@\${SERVER_IP}:/home/ec2-user
                        ssh -i \$MY_SSH_KEY -o StrictHostKeyChecking=no \${USERNAME}@\${SERVER_IP} /bin/bash << 'EOF'
                            cd /home/ec2-user
                            unzip -o myapp.zip -d app
                            . app/venv/bin/activate || python3 -m venv app/venv && . app/venv/bin/activate
                            cd app
                            pip install --upgrade pip
                            pip install -r requirements.txt
                            sudo systemctl restart flaskapp.service
                        EOF
                    """
                }
            }
        }
    }
}

