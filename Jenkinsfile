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
                // Call the scmSkipCI function after checkout
                scmSkipCI(deleteBuild: true)
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

    // Define skip patterns
    def skipPatterns = ['.*\\[ci skip\\].*', '.*ci skip.*']

    // Get the latest commit message
    def commitMessage = getCommitMessage()
    echo "Latest commit message: ${commitMessage}"

    // Check if the commit message matches any of the skip patterns
    for (pattern in skipPatterns) {
        if (commitMessage ==~ /${pattern}/) {
            echo "Skipping build due to commit message matching pattern: ${pattern}"
            if (deleteBuild) {
                currentBuild.rawBuild.delete()  // Permanently delete the build
                error("Build skipped and deleted due to SCM skip pattern") // Optional error message
            }
        }
    }
    echo "Proceeding with the build"
}
