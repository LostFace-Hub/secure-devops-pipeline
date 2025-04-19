pipeline {
    agent any
    environment {
        DOCKER_BUILDKIT = 1
        DOCKER_CLI_EXPERIMENTAL = 1
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Ensure Docker is working fine
                    sh 'docker --version'
                    sh 'docker build -t secure-devops-project .'
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                script {
                    // Ensure trivy is installed and working
                    sh 'trivy --version'
                    sh 'trivy image --timeout 600s --format table --output trivy-report/scan.txt secure-devops-project'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
