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
                pip install pytest pip-audit
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

        stage('SCA Scan (pip-audit)') {
            steps {
                sh '''
                . venv/bin/activate
                pip-audit -r requirements.txt
                '''
            }
        }

        stage('SCA Scan (Dependency-Check)') {
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
                    --out dependency-check-report
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