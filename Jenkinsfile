pipeline {
    agent any

    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                pip install pytest semgrep
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                . venv/bin/activate
                pytest
                '''
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
                    --exclude './venv/**' \
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
                sh '''
                . venv/bin/activate
                semgrep scan --config auto . || true
                '''
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