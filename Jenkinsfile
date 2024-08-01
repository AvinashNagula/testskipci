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
                        return shouldSkipBuild()
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

// Function to check if the build should be skipped based on commit messages
def shouldSkipBuild() {
    def changeSets = currentBuild.changeSets
    if (changeSets && changeSets.size() > 0) {
        def commitMessages = changeSets.collectMany { it.items.collect { it.msg } }
        return commitMessages.any { it.matches('.*\\[ci skip\\].*') }
    }
    return false
}
