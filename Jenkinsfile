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
                    // Checkout code from the specified Git repository
                    def changelog = checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']], // Checkout the 'main' branch
                        userRemoteConfigs: [[url: 'https://github.com/AvinashNagula/testskipci.git']] // Your Git URL
                    ])

                    // Check if any commit message contains "[ci skip]"
                    if (shouldSkipBuild(changelog)) {
                        currentBuild.result = 'ABORTED'
                        error('Skipping build due to "[ci skip]" in commit message')
                    }
                }
            }
        }
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}

// Function to check for skip CI pattern in commit messages
def shouldSkipBuild(changelog) {
    def skipPattern = ~/.*\[ci skip\].*/ // Pattern to look for

    // Check if changelog is not null and contains entries
    if (changelog && changelog.get("git") && changelog.get("git").get("branches")) {
        def messages = changelog.get("git").get("branches").collect { branch ->
            branch.get("commit")?.get("message") // Safely extract commit message
        }.findAll { it } // Filter out null messages

        return messages.any { msg -> skipPattern.matcher(msg).matches() } // Check for matches
    }
    return false // Return false if changelog is null or empty
}
