pipeline {
    agent any

    options {
        skipDefaultCheckout true
    }
    stages {
        stage('Checkout') {
            when {
                not {
                    expression {
                        def changeSets = currentBuild.changeSets
                        if (changeSets && changeSets.size() > 0) {
                            def commitMessages = changeSets.collectMany { it.items.collect { it.msg } }
                            return commitMessages.any { it.matches('.*\\[ci skip\\].*') }
                        }
                        return false
                    }
                }
            }
            steps {
                // Checkout code from a Git repository
                git url: 'https://github.com/your-repo/your-project.git',
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