pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from SCM
                checkout scm

                // Skip build if commit message contains 'ci skip' or 'skip ci' in any form
                scmSkip(deleteBuild: true, skipPattern: '.*ci skip.*|.*skip ci.*')
            }
        }

        stage('Build') {
            when {
                not {
                    // Skip the build stage if scmSkip has marked the build for skipping
                    environment name: 'SKIP_BUILD', value: 'true'
                }
            }
            steps {
                echo 'Building...'
                // Add your build steps here, e.g., compile code, run tests, etc.
            }
        }

        stage('Test') {
            when {
                not {
                    // Skip the test stage if scmSkip has marked the build for skipping
                    environment name: 'SKIP_BUILD', value: 'true'
                }
            }
            steps {
                echo 'Testing...'
                // Add your test steps here
            }
        }

        stage('Deploy') {
            when {
                not {
                    // Skip the deploy stage if scmSkip has marked the build for skipping
                    environment name: 'SKIP_BUILD', value: 'true'
                }
            }
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
