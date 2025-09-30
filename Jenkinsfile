pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }

    stages {

        stage('Set Up') {
            steps {
                sh '''
                    #!/bin/bash
                    set -e
                    python3 -m venv venv
                    venv/bin/python -m pip install --upgrade pip
                    venv/bin/python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    #!/bin/bash
                    set -e
                    venv/bin/python -m pytest
                '''
            }
        }

        stage('Package code') {
            steps {
                sh '''
                    #!/bin/bash
                    set -e
                    # Install zip if missing
                    command -v zip >/dev/null 2>&1 || sudo yum install -y zip
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
                        scp -i "$MY_SSH_KEY" -o StrictHostKeyChecking=no myapp.zip ${USERNAME}@${SERVER_IP}:/home/ec2-user
                        ssh -i "$MY_SSH_KEY" -o StrictHostKeyChecking=no ${USERNAME}@${SERVER_IP} /bin/bash << 'EOF'
                            set -e
                            cd /home/ec2-user
                            unzip -o myapp.zip -d app
                            # Ensure virtualenv exists
                            if [ ! -d "app/venv" ]; then
                                python3 -m venv app/venv
                            fi
                            # Use venv Python and pip directly
                            app/venv/bin/python -m pip install --upgrade pip
                            app/venv/bin/python -m pip install -r app/requirements.txt
                            sudo systemctl restart flaskapp.service
                        EOF
                    '''
                }
            }
        }
    }
}
