pipeline {
    agent any

    stages {
        stage('Install Dependencies') {
            steps {
                sh 'python3 -m pip install --break-system-packages -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'python3 -m pytest'
            }
        }

        stage('SCA Scan') {
            steps {
                sh '''
                dependency-check.sh \
                  --project "TP-Jenkins" \
                  --scan . \
                  --format HTML \
                  --out dependency-check-report \
                  --failOnCVSS 7
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'dependency-check-report/**', fingerprint: true
                }
            }
        }

        stage('SAST Scan') {
            steps {
                sh 'sonar-scanner'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
        }
        failure {
            echo 'Build failed due to errors or vulnerabilities'
        }
    }
}