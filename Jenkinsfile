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
                sh ". venv/bin/activate && pytest"
            }
        }

        stage('Package code') {
            steps {
                sh '''
                    #!/bin/bash
                    set -e
                    # Install zip if missing
                    command -v zip >/dev/null 2>&1 || apt install -y zip
                    # Package everything EXCEPT the Jenkins venv
                    zip -r myapp.zip . -x '*.git*' -x 'venv/*'
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
                        scp -i "$MY_SSH_KEY" -o StrictHostKeyChecking=no myapp.zip ${USERNAME}@${SERVER_IP}:/home/ubuntu
                        ssh -i "$MY_SSH_KEY" -o StrictHostKeyChecking=no ${USERNAME}@${SERVER_IP} /bin/bash << 'EOF'
                            set -e
                            commd -v unzip >/dev/null 2>&1 || sudo apt install -y unzip
                            unzip -o /home/ubuntu/myapp.zip -d /home/ubuntu/app
                            # Ensure virtualenv exists
                            if [ ! -d "app/venv" ]; then
                                python3 -m venv venv
                            fi
                            source app/venv/bin/activate
                            cd /home/ubuntu/app
                            # Install requirements using the venv pip
                            pip install -r requirements.txt
                            sudo systemctl restart flaskapp.service
                        EOF
                    '''
                }
            }
        }
    }
}
