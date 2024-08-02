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
                    try {
                        // Checkout code from a Git repository
                        git url: 'https://github.com/AvinashNagula/testskipci.git', branch: 'main'
                        scmSkipCI(deleteBuild: false, skipPattern: '.*ci skip.*')
                    } catch (hudson.AbortException e) {
                        // Handle build skipping
                        echo 'Build skipping initiated'
                        throw e
                    }
                }
            }
        }
        stage('Build') {
            when {
                not {
                    equals expected: 'NOT_BUILT', actual: currentBuild.result
                }
            }
            steps {
                // Add your build steps here
                echo 'Building....'
            }
        }
        stage('Test') {
            when {
                not {
                    equals expected: 'NOT_BUILT', actual: currentBuild.result
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
                echo "Build skipped due to commit message matching skip pattern."
            }
        }
    }
}

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

def getCommitMessage() {
    // This assumes the use of the Git plugin
    def commitMessage = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
    return commitMessage
}
