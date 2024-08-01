pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from SCM
                git url: 'https://github.com/AvinashNagula/testskipci.git',
                    branch: 'main'

                script {
                    // Skip build if commit message contains 'ci skip' or 'skip ci' in any form
                    def skipPattern = '.*ci skip.*|.*skip ci.*'
                    if (shouldSkipBuild(skipPattern)) {
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

// Define the function to check commit messages
def shouldSkipBuild(String skipPattern) {
    def changeSets = currentBuild.changeSets
    def shouldSkip = false

    if (changeSets == null) {
        echo "No change sets found."
        return false
    }

    changeSets.each { changeSet ->
        if (changeSet.items == null) {
            echo "Change set items are null."
            return false
        }

        changeSet.items.each { entry ->
            def message = entry.msg
            if (message == null) {
                echo "Commit message is null."
                return false
            }
            if (message ==~ skipPattern) {
                echo "Skipping build due to commit message: ${message}"
                shouldSkip = true
            }
        }
    }

    return shouldSkip
}

// Call the function at the end of the Jenkinsfile to check for skip pattern
if (shouldSkipBuild('.*ci skip.*|.*skip ci.*')) {
    currentBuild.result = 'NOT_BUILT'
    error("Build skipped due to presence of skip token in commit message")
}
