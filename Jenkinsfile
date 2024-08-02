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
                // Checkout code from a Git repository
                git url: 'https://github.com/AvinashNagula/testskipci.git', branch: 'main'
                scmSkipCI(deleteBuild: true, skipPattern: '.*\\[ci skip\\].*')

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
