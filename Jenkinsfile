pipeline {
    agent any

    stages {
        stage('Checkout') {
            when {
                not {
                    expression {
                        return env.CHANGE_MESSAGE.matches('.*\\[ci skip\\].*')
                    }
                }
            }
            steps {
                // Checkout code from a Git repository
                git url: 'https://github.com/AvinashNagula/testskipci.git',
                    branch: 'main'
            }
        }
    }
}

// Define the function to check commit messages
def shouldSkipBuild(String skipToken = '[ci skip]') {
    def changeSets = currentBuild.changeSets
    def shouldSkip = false

    changeSets.each { changeSet ->
        changeSet.items.each { entry ->
            def message = entry.msg
            if (message.contains(skipToken)) {
                echo "Skipping build due to commit message: ${message}"
                shouldSkip = true
            }
        }
    }

    return shouldSkip
}

// Call the function at the end of the Jenkinsfile
if (shouldSkipBuild('[ci skip]')) {
    currentBuild.result = 'NOT_BUILT'
    error("Build skipped due to presence of skip token in commit message")
}
