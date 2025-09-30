pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }

    stages {

        stage('Set Up') {
            steps {                
                sh "pip install -r requirements.txt"
            }
        }

        stage('Test') {
            steps {
                sh "pytest"
            }
        }

        stage('Package code') {
            steps {
                sh '''
                    #!/bin/bash
                    set -e
                    # Install zip if missing
                    command -v zip >/dev/null 2>&1 || yum install -y zip
                    # Package everything EXCEPT the Jenkins venv
                    zip -r myapp.zip . -x '*.git*'
                    ls -lart
                '''
            }
        }

        stage('Deploy to prod') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ssh-key',
                    keyFileVariable: 'MY_SSH_KEY',
                    usernameVariable: 'USERNAME'
                )]) {
                    sh '''
                        #!/bin/bash
                        set -e
                        scp -i "$MY_SSH_KEY" -o StrictHostKeyChecking=no myapp.zip ${USERNAME}@${SERVER_IP}:/home/ec2-user
                        ssh -i "$MY_SSH_KEY" -o StrictHostKeyChecking=no ${USERNAME}@${SERVER_IP} /bin/bash << 'REMOTE_EOF'
                            set -e
                            commd -v unzip >/dev/null 2>&1 || sudo yum install -y unzip
                            unzip -o /home/ec2-user/myapp.zip -d /home/ec2-user/app
                            # Ensure virtualenv exists
                            if [ ! -d "app/venv" ]; then
                                python3 -m venv venv
                            fi
                            source app/venv/bin/activate
                            cd /home/ec2-user/app
                            # Install requirements using the venv pip
                            sudo yum install -y python3-pip
                            python3 -m pip install -r requirements.txt
                            sudo systemctl restart flaskapp.service || echo 'Failed to restart service'

REMOTE_EOF
                    '''
                }
            }
        }
    }
}
