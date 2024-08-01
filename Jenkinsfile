pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from the specified Git repository and branch
                git url: 'https://github.com/AvinashNagula/testskipci.git', branch: 'main'
            }
        }

        stage('Check for Skip CI') {
            steps {
                script {
                    // Define the skip pattern to look for in the commit message
                    def skipPattern = '.*ci skip.*|.*skip ci.*'
                    def latestCommitMessage = getLatestCommitMessage()

                    // Check the latest commit message against the skip pattern
                    if (latestCommitMessage != null && latestCommitMessage ==~ skipPattern) {
                        env.SKIP_BUILD = 'true'
                        currentBuild.result = 'NOT_BUILT' // Mark the build as not built

                        // Delete the current build
                        deleteCurrentBuild()
                    } else {
                        env.SKIP_BUILD = 'false'
                    }
                }
            }
        }

        stage('Build') {
            when {
                expression { env.SKIP_BUILD != 'true' }
            }
            steps {
                echo 'Building...'
                // Add your build steps here, e.g., compile code, run tests, etc.
            }
        }

        stage('Test') {
            when {
                expression { env.SKIP_BUILD != 'true' }
            }
            steps {
                echo 'Testing...'
                // Add your test steps here
            }
        }

        stage('Deploy') {
            when {
                expression { env.SKIP_BUILD != 'true' }
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

// Define the function to get the latest commit message
def getLatestCommitMessage() {
    def changeSets = currentBuild.changeSets

    if (changeSets == null || changeSets.isEmpty()) {
        echo "No change sets found."
        return null
    }

    def latestChangeSet = changeSets[-1] // Get the latest change set
    if (latestChangeSet.items == null || latestChangeSet.items.length == 0) {
        echo "Change set items are null or empty."
        return null
    }

    def latestCommit = latestChangeSet.items[-1] // Get the latest commit
    return latestCommit.msg
}

// Define the function to delete the current build without using curl
def deleteCurrentBuild() {
    script {
        // Get the current job and build
        def job = Jenkins.instance.getItemByFullName(env.JOB_NAME)
        def build = job.getBuildByNumber(env.BUILD_NUMBER as int)

        // Delete the current build
        build.delete()
        echo "Build ${env.BUILD_NUMBER} deleted due to [skip ci] in commit message."
    }
}
