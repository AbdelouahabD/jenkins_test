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

        stage('Python Dependency Scan (pip-audit)') {
            steps {
                sh '''
                python3 -m venv audit-venv
                . audit-venv/bin/activate
                pip install --upgrade pip
                pip install pip-audit
                pip-audit -r requirements.txt
                '''
            }
        }

        stage('SCA Scan (Dependency-Check)') {
            steps {
                script {
                    def dcTool = tool 'dependency-check'
                    sh "${dcTool}/bin/dependency-check.sh --project 'TP-Jenkins' --scan . --format HTML --out dependency-check-report --failOnCVSS 7"
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'dependency-check-report/**', fingerprint: true
                }
            }
        }

        stage('SAST Scan (SonarCloud)') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarCloud') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=abdelouahabd_jenkins_test \
                        -Dsonar.organization=abdelouahabd \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.exclusions=dependency-check-report/** \
                        -Dsonar.python.version=3.13
                        """
                    }
                }
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