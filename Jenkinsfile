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
                    source venv/bin/activate
                    python3 -m pip install --upgrade pip
                    python3 -m pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    source venv/bin/activate
                    python3 -m pytest
                '''
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
                        ssh -i \$MY_SSH_KEY -o StrictHostKeyChecking=no \${USERNAME}@\${SERVER_IP} /bin/bash << EOF
                            set -e
                            cd /home/ec2-user
                            unzip -o myapp.zip -d app

                            # create venv if it doesn't exist
                            if [ ! -d app/venv ]; then
                                python3 -m venv app/venv
                            fi

                            # activate venv and install dependencies
                            source app/venv/bin/activate
                            python3 -m pip install --upgrade pip
                            python3 -m pip install -r app/requirements.txt

                            # restart the flask service
                            sudo systemctl restart flaskapp.service
EOF
                    """
                }
            }
        }

    }
}

