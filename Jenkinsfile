pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from SCM
                checkout scm

                // Skip build if commit message contains [ci skip] or [skip ci]
                scmSkip(deleteBuild: true, skipPattern: '.*\\[ci skip\\].*|.*\\[skip ci\\].*')
            }
        }

        stage('Build') {
            steps {
                echo 'Building...'
                // Add your build steps here, e.g., compile code, run tests, etc.
            }
        }

        stage('Test') {
            steps {
                echo 'Testing...'
                // Add your test steps here
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Add your deployment steps here
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            // Add any cleanup steps here
        }

        success {
            echo 'Build succeeded!'
        }

        failure {
            echo 'Build failed!'
        }
    }
}
