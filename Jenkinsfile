pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run tests') {
            steps {
                sh 'npm test'
            }
        }
    }

    post {
        always {
            junit '**/test-results/*.xml' // si tu génères des rapports JUnit
        }
        success {
            echo 'Pipeline réussie !'
        }
        failure {
            echo 'Pipeline échouée.'
        }
    }
}