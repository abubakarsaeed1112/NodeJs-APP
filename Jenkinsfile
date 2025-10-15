pipeline {
    agent any

    environment {
        DEV_KEY = credentials('dev-key')
        STAGE_KEY = credentials('stage-key')
        PROD_KEY = credentials('prod-key')
        GITHUB_TOKEN = credentials('abubakarsaeed1112')
        APP_DIR = "/home/ubuntu/nodejs-express-template"
        APP_PORT = "3000"
    }

    stages {
        stage('Test Run') {
            steps {
                echo 'Hello Jenkins, running local test!'
            }
        }

        stage('Build & Install') {
            steps {
                echo 'Installing dependencies...'
                sh "cd ${APP_DIR} && npm install"
            }
        }

        stage('Pre-Deploy Input') {
            steps {
                input message: 'Select deployment server', parameters: [
                    choice(name: 'SERVER', choices: ['dev','stage','prod'], description: 'Which server to deploy?')
                ]
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def key
                    if (params.SERVER == 'dev') { key = DEV_KEY }
                    else if (params.SERVER == 'stage') { key = STAGE_KEY }
                    else { key = PROD_KEY }

                    def host = params.SERVER == 'dev' ? 'DEV_EC2_IP' : params.SERVER == 'stage' ? 'STAGE_EC2_IP' : 'PROD_EC2_IP'

                    sh """
                        ssh -o StrictHostKeyChecking=no -i ${key} ubuntu@${host} '
                        cd ${APP_DIR} || mkdir -p ${APP_DIR}
                        git pull origin main
                        npm install
                        pm2 stop app || true
                        pm2 start index.js --name app --port ${APP_PORT}
                        '
                    """
                }
            }
        }

    }

    post {
        success {
            echo "Deployment to ${params.SERVER} completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
