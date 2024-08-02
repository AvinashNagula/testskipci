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
            steps {
                script {
                    // Checkout code from a Git repository
                    git url: 'https://github.com/AvinashNagula/testskipci.git', branch: 'main'

                    // Call the SCM Skip CI function
                    scmSkipCI(deleteBuild: true, skipPattern: '.*\\[ci skip\\].*')
                }
            }
        }
        stage('Build') {
            when {
                not {
                    expression { currentBuild.result == 'NOT_BUILT' }
                }
            }
            steps {
                // Add your build steps here
                echo 'Building...'
            }
        }
        stage('Test') {
            when {
                not {
                    expression { currentBuild.result == 'NOT_BUILT' }
                }
            }
            steps {
                // Add your test steps here
                echo 'Testing...'
            }
        }
    }

    post {
        always {
            script {
                if (currentBuild.result == 'ABORTED') {
                    echo "Build was aborted due to 'ci skip' in the commit message."
                    deleteBuild() // Call the delete build method
                }
            }
        }
    }
}

// Function to check if the build should be skipped based on the commit message
def scmSkipCI(Map params = [:]) {
    def deleteBuild = params.get('deleteBuild', false)
    def skipPattern = params.get('skipPattern', '.*\\[ci skip\\].*')

    def commitMessage = getCommitMessage()
    if (commitMessage ==~ /${skipPattern}/) {
        echo "Skipping build due to commit message matching pattern ${skipPattern}"
        if (deleteBuild) {
            currentBuild.result = 'NOT_BUILT'
            throw new hudson.AbortException("Build skipped due to SCM skip pattern")
        }
    } else {
        echo "Proceeding with the build"
    }
}

// Function to get the commit message
def getCommitMessage() {
    // This assumes the use of the Git plugin
    def commitMessage = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
    return commitMessage
}

// Function to delete the build
def deleteBuild() {
    echo "Deleting the build marked as ABORTED."
    currentBuild.rawBuild.delete()
}
