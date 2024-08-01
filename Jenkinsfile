import hudson.model.Run
import hudson.model.TaskListener
import hudson.scm.ChangeLogSet
import jenkins.scm.RunWithSCM
import java.util.logging.Logger
import java.util.regex.Pattern

// Logger for debugging
def LOGGER = Logger.getLogger(this.class.name)

// Function to check if the build should be skipped based on commit messages
def shouldSkipBuild(Run<?, ?> run, Pattern pattern, TaskListener listener) {
    def changeSets = run instanceof RunWithSCM ? ((RunWithSCM<?, ?>) run).getChangeSets() : []
    if (changeSets.isEmpty()) {
        listener.getLogger().println("SCM Skip: Changelog is empty!")
        LOGGER.fine("Changelog is empty!")
        return false
    }

    def changeLogSet = changeSets.last()
    if (changeLogSet.isEmptySet()) {
        listener.getLogger().println("SCM Skip: Changelog is empty!")
        LOGGER.fine("Changelog is empty!")
        return false
    }

    def allSkipped = true
    def notMatched = ""

    for (ChangeLogSet.Entry entry : changeLogSet) {
        def fullMessage = getFullMessage(entry)
        if (!pattern.matcher(fullMessage).matches()) {
            allSkipped = false
            notMatched = fullMessage
            break
        }
    }

    def commitMessage = combineChangeLogMessages(changeLogSet)
    def logMessage = allSkipped ?
        "SCM Skip: Pattern ${pattern.pattern()} matched on message: ${commitMessage}" :
        "SCM Skip: Pattern ${pattern.pattern()} NOT matched on message: ${notMatched}"

    listener.getLogger().println(logMessage)
    LOGGER.fine(logMessage)

    return allSkipped
}

// Helper function to combine change log messages
def combineChangeLogMessages(ChangeLogSet<?> changeLogSet) {
    return changeLogSet.collect { getFullMessage(it) }.join(" ")
}

// Helper function to get the full message from a change log entry
def getFullMessage(ChangeLogSet.Entry entry) {
    if (Jenkins.getInstance().getPlugin("git") != null) {
        return GitMessageExtractor.getFullMessage(entry)
    }
    return entry.getMsg()
}

// Example usage in a Jenkins pipeline
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
                        def pattern = ~/.*\[ci skip\].*/
                        return shouldSkipBuild(currentBuild.rawBuild, pattern, currentBuild.rawBuild.getListener())
                    }
                }
            }
            steps {
                // Checkout code from a Git repository
                git url: 'https://github.com/AvinashNagula/testskipci.git', branch: 'main'
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
