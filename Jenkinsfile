pipeline {
    agent any

    options {
        skipDefaultCheckout true
    }

    triggers {
        pollSCM 'H/5 * * * *' // Poll SCM every 5 minutes
    }

    stages {
        stage('Checkout') {
            when {
                not {
                    expression {
                        return env.CHANGE_MESSAGE.matches('.*\\[ci skip\\].*')
                    }
                }
            }
            steps {
                // Checkout code from a Git repository
                git url: 'https://github.com/AvinashNagula/testskipci.git',
                    branch: 'main'
            }
        }
        stage('Build') {
            steps {
                // Add your build steps here
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                // Add your test steps here
                echo 'Testing...'
            }
        }
        stage('Deploy') {
            steps {
                // Add your deployment steps here
                echo 'Deploying...'
            }
        }
    }
}