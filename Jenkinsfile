pipeline {
    agent any

    environment {
        APP_SERVER = '172.31.36.107'
        APP_DIR = '/opt/webapp'
        APP_PORT = '3000'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Package') {
            steps {
                sh '''
                    tar -czf webapp.tar.gz \
                        app.js \
                        package.json \
                        package-lock.json
                '''
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['app-server-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} \
                            "mkdir -p ${APP_DIR}"

                        scp -o StrictHostKeyChecking=no \
                            webapp.tar.gz \
                            ubuntu@${APP_SERVER}:${APP_DIR}/

                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} "
                            cd ${APP_DIR} &&
                            tar -xzf webapp.tar.gz &&
                            npm install --omit=dev &&
                            sudo systemctl restart webapp
                        "
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sshagent(['app-server-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} \
                            "curl -f http://localhost:${APP_PORT}/health"
                    '''
                }
            }
        }
    }
}
