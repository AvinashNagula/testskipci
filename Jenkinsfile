pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from the specified Git repository and branch
                git url: 'https://github.com/AvinashNagula/testskipci.git', branch: 'main'

                script {
                    // Skip build if the latest commit message contains 'ci skip' or 'skip ci' in any form
                    def skipPattern = '.*ci skip.*|.*skip ci.*'
                    def latestCommitMessage = getLatestCommitMessage()
                    if (latestCommitMessage != null && latestCommitMessage ==~ skipPattern) {
                        env.SKIP_BUILD = 'true'
                    } else {
                        env.SKIP_BUILD = 'false'
                    }
                }
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

// Define the function to get the latest commit message
def getLatestCommitMessage() {
    def changeSets = currentBuild.changeSets

    if (changeSets == null || changeSets.isEmpty()) {
        echo "No change sets found."
        return null
    }

    def latestChangeSet = changeSets[-1] // Get the latest change set
    if (latestChangeSet.items == null || latestChangeSet.items.isEmpty()) {
        echo "Change set items are null or empty."
        return null
    }

    def latestCommit = latestChangeSet.items[-1] // Get the latest commit
    return latestCommit.msg
}

// Call the function at the end of the Jenkinsfile to check for skip pattern in the latest commit message
def skipPattern = '.*ci skip.*|.*skip ci.*'
def latestCommitMessage = getLatestCommitMessage()
if (latestCommitMessage != null && latestCommitMessage ==~ skipPattern) {
    currentBuild.result = 'NOT_BUILT'
    error("Build skipped due to presence of skip token in the latest commit message")
}
