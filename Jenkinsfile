pipeline {
    agent {
        docker {
            image 'python:3.13'
        }
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                    # Hna kantsabou l-bit (venv) bach n-tfadaw dak l-erreur
                    python3 -m venv venv
                    ./venv/bin/pip install --upgrade pip
                    ./venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                # Khassek t-dir ./venv/bin/ bach t-khdem b pytest lli installiti
                sh './venv/bin/pytest'
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
        failure {
            echo 'Build failed due to errors or vulnerabilities'
        }
    }
}