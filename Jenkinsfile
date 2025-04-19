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
                    bat "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    bat "if not exist trivy-report mkdir trivy-report"
                    bat "trivy image --format table --output ${REPORT_PATH} ${IMAGE_NAME}"
                }
            }
        }

        stage('Check Vulnerabilities') {
            steps {
                script {
                    def report = readFile("${REPORT_PATH}")
                    if (report.contains("CRITICAL") || report.contains("HIGH")) {
                        echo "High or Critical vulnerabilities found!"
                        currentBuild.result = 'FAILURE'
                        error("Aborting due to vulnerabilities.")
                    } else {
                        echo "No major vulnerabilities found. Safe to proceed."
                    }
                }
            }
        }

        stage('Deploy (Docker Compose)') {
            steps {
                script {
                    bat 'docker-compose down || true'
                    bat 'docker-compose up -d'
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'trivy-report/*.txt', allowEmptyArchive: true
        }

        failure {
            echo "Build failed due to vulnerabilities or other errors."
        }
    }
}
