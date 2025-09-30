
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
                    python -m pip install --upgrade pip
                    python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    python -m pytest
                '''
            }
        }

        stage('Package code') {
            steps {
                sh '''
                    sudo apt-get update || true
                    sudo apt-get install -y zip || true
                    zip -r myapp.zip ./* -x '*.git*'
                    ls -lart
                '''
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

                            # create venv if not exists
                            if [ ! -d app/venv ]; then
                                python3 -m venv app/venv
                            fi

                            . app/venv/bin/activate
                            python -m pip install --upgrade pip
                            python -m pip install -r app/requirements.txt

                            sudo systemctl restart flaskapp.service
                        EOF
                    """
                }
            }
        }
    }
}

