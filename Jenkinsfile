pipeline {
    agent any

    stages {
        stage('Install App Dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                pip install pytest
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
                    --exclude './sast-venv/**' \
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
                python3 -m venv sast-venv
                . sast-venv/bin/activate
                pip install --upgrade pip
                pip install semgrep
                semgrep scan --config auto --exclude venv --exclude sast-venv --exclude dependency-check-report .
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