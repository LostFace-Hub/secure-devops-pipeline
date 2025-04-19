pipeline {
    agent any

    environment {
        IMAGE_NAME = "secure-flask-app"
        REPORT_PATH = "trivy-report/scan.txt"
    }

    stages {
        stage('Clone') {
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t ${env.IMAGE_NAME} ."
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    bat 'if not exist trivy-report mkdir trivy-report'
                    bat "trivy image --format table --output ${env.REPORT_PATH} ${env.IMAGE_NAME}"
                }
            }
        }

        stage('Check Vulnerabilities') {
            steps {
                script {
                    def report = readFile("${env.REPORT_PATH}")
                    if (report.contains("CRITICAL") || report.contains("HIGH")) {
                        error("High or Critical vulnerabilities found! Aborting.")
                    } else {
                        echo "No major vulnerabilities found. Safe to proceed."
                    }
                }
            }
        }

        stage('Deploy (Docker Compose)') {
            steps {
                bat 'docker-compose down || true'
                bat 'docker-compose up -d'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'trivy-report/*.txt', allowEmptyArchive: true
        }
    }
}
