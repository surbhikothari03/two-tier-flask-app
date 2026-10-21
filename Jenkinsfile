pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ---------- DEV (auto deploy) ----------
        stage('Deploy to DEV') {
            when { branch 'dev' }
            environment {
                FLASK_PORT = '3000'
            }
            steps {
                sh '''
                    docker compose -p twotier-dev -f docker-compose.ci.yml up -d --build

                    for i in $(seq 1 20); do
                        if curl -sf http://localhost:$FLASK_PORT/ > /dev/null; then
                            echo "DEV app is UP"
                            exit 0
                        fi
                        echo "waiting for app... $i"
                        sleep 5
                    done

                    docker compose -p twotier-dev -f docker-compose.ci.yml logs --tail 50
                    exit 1
                '''
            }
        }

        // ---------- PROD (approval ke baad) ----------
        stage('Approval for PROD') {
            when { branch 'master' }
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input message: 'Production par deploy karna hai?',
                          ok: 'Yes, Deploy',
                          submitter: 'azureuser'
                }
            }
        }

        stage('Deploy to PROD') {
            when { branch 'master' }
            environment {
                FLASK_PORT = '4000'
            }
            steps {
                sh '''
                    docker compose -p twotier-prod -f docker-compose.ci.yml up -d --build

                    for i in $(seq 1 20); do
                        if curl -sf http://localhost:$FLASK_PORT/ > /dev/null; then
                            echo "PROD app is UP"
                            exit 0
                        fi
                        echo "waiting for app... $i"
                        sleep 5
                    done

                    docker compose -p twotier-prod -f docker-compose.ci.yml logs --tail 50
                    exit 1
                '''
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f'
        }
        success { echo 'Pipeline successful!' }
        failure { echo 'Pipeline failed, Console Output check karo.' }
    }
}
