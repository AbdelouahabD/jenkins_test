pipeline {
    agent any

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m pip install --upgrade pip
                python3 -m pip install -r requirements.txt
                python3 -m pip install pytest semgrep
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh 'python3 -m pytest'
            }
        }

        stage('SCA Scan') {
            steps {
                script {
                    def dcTool = tool 'dependency-check'
                    sh """
                    ${dcTool}/bin/dependency-check.sh \
                    --project 'TP-Jenkins' \
                    --scan . \
                    --exclude './audit-venv/**' \
                    --exclude './dependency-check-report/**' \
                    --format HTML \
                    --out dependency-check-report \
                    --failOnCVSS 7
                    """
                }
            }
        }

        stage('SAST Scan') {
            steps {
                sh 'semgrep scan --config auto . || true'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'dependency-check-report/**', fingerprint: true
        }
        failure {
            echo 'Build failed due to errors or vulnerabilities'
        }
    }
}