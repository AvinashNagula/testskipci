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
                        return shouldSkipBuild(currentBuild.rawBuild, pattern)
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

package net.plavcak.jenkins.plugins.scmskip

import hudson.AbortException
import hudson.model.AbstractBuild
import hudson.model.AbstractProject
import hudson.model.Cause
import hudson.model.Job
import hudson.model.Result
import hudson.model.Run
import hudson.model.TaskListener
import hudson.scm.ChangeLogSet
import java.io.IOException
import java.util.List
import java.util.logging.Level
import java.util.logging.Logger
import javax.servlet.ServletException
import jenkins.model.CauseOfInterruption
import jenkins.model.Jenkins
import jenkins.scm.RunWithSCM
import org.jenkinsci.plugins.workflow.job.WorkflowRun
import org.jenkinsci.plugins.workflow.steps.FlowInterruptedException

class SCMSkipTools {

    private static final Logger LOGGER = Logger.getLogger(SCMSkipTools.class.name)

    private SCMSkipTools() {
        // Utility class
    }

    /**
     * Deletes given build.
     * @param build Build to delete
     * @throws IOException if build cannot be deleted
     */
    static void deleteBuild(AbstractBuild<?, ?> build) throws IOException {
        LOGGER.log(Level.FINE, "Deleting AbstractBuild: '${build.getId()}'")
        build.delete()
        AbstractProject<?, ?> project = build.getProject()
        project.updateNextBuildNumber(build.getNumber())
        project.save()
    }

    /**
     * Deletes given run.
     * @param run Run to delete
     * @throws IOException if run cannot be deleted
     */
    static void deleteRun(Run<?, ?> run) throws IOException {
        LOGGER.log(Level.FINE, "Deleting Run: '${run.getId()}'")
        run.delete()
        Job<?, ?> job = run.getParent()
        job.updateNextBuildNumber(run.number)
        job.save()
    }

    static void tagRunForDeletion(Run<?, ?> run, boolean deleteBuild) throws IOException {
        LOGGER.log(Level.FINE, "Build '${run.getDisplayName()}' set to delete: ${deleteBuild}")
        run.addAction(new SCMSkipDeleteEnvironmentAction(deleteBuild))
        run.save()
    }

    static boolean isBuildToDelete(Run<?, ?> run) {
        SCMSkipDeleteEnvironmentAction action = run.getAction(SCMSkipDeleteEnvironmentAction.class)
        return action != null && action.isDeleteBuild()
    }

    /**
     * Inspect build for matched pattern in changelog and user cause.
     * @param run Current build
     * @param matcher Matcher object
     * @param listener Runtime listener
     * @return True if at least one entry matched and build not started by user
     */
    static boolean inspectChangeSetAndCause(Run<?, ?> run, SCMSkipMatcher matcher, TaskListener listener) {
        if (run.getCauses().any { it instanceof Cause.UserIdCause }) {
            return false
        }
        if (run instanceof RunWithSCM) {
            return inspectChangeSet(((RunWithSCM<?, ?>) run).getChangeSets(), matcher, listener.getLogger())
        }
        return false
    }

    private static boolean inspectChangeSet(
            List<ChangeLogSet<? extends ChangeLogSet.Entry>> changeLogSets,
            SCMSkipMatcher matcher,
            PrintStream logger) {
        if (changeLogSets.isEmpty()) {
            logEmptyChangeLog(logger)
            return false
        }

        ChangeLogSet<? extends ChangeLogSet.Entry> changeLogSet = changeLogSets.last()

        if (changeLogSet.isEmptySet()) {
            logEmptyChangeLog(logger)
        }

        boolean allSkipped = true
        String notMatched = ""

        changeLogSet.each { entry ->
            String fullMessage = getFullMessage(entry)
            if (!matcher.match(fullMessage)) {
                allSkipped = false
                notMatched = fullMessage
                return // Exit the loop early
            }
        }

        String commitMessage = combineChangeLogMessages(changeLogSet)

        String logMessage
        if (!allSkipped) {
            logMessage = "SCM Skip: Pattern ${matcher.getPattern().pattern()} NOT matched on message: ${notMatched}"
        } else {
            logMessage = "SCM Skip: Pattern ${matcher.getPattern().pattern()} matched on message: ${commitMessage}"
        }
        logger.println(logMessage)
        LOGGER.log(Level.FINE, logMessage)

        return allSkipped
    }

    /**
     * @param run Run to be terminated
     * @throws IOException May throw AbortException specifically to terminate build
     * @throws ServletException When build stopping fails
     * @throws FlowInterruptedException To terminate pipeline build
     */
    static void stopBuild(Run<?, ?> run) throws IOException, ServletException, FlowInterruptedException {
        run.setDescription("SCM Skip - build skipped")
        run.setResult(Result.ABORTED)
        run.save()

        LOGGER.log(Level.FINE, "Stopping build: '${run.getId()}'")

        if (run instanceof WorkflowRun) {
            throw new FlowInterruptedException(Result.NOT_BUILT, true, new CauseOfInterruption() {
                @Override
                String getShortDescription() {
                    return "Skipped because of SCM message"
                }
            })
        } else if (run instanceof AbstractBuild) {
            AbstractBuild<?, ?> build = (AbstractBuild<?, ?>) run
            build.doStop()
        } else {
            throw new AbortException("SCM Skip: Build has been skipped due to SCM Skip Plugin!")
        }
    }

    private static String combineChangeLogMessages(ChangeLogSet<?> changeLogSet) {
        return changeLogSet.collect { getFullMessage(it) }.join(" ")
    }

    private static void logEmptyChangeLog(PrintStream logger) {
        logger.println("SCM Skip: Changelog is empty!")
        LOGGER.log(Level.FINE, "Changelog is empty!")
    }

    private static String getFullMessage(ChangeLogSet.Entry entry) {
        if (Jenkins.get().getPlugin("git") != null) {
            return GitMessageExtractor.getFullMessage(entry)
        }
        return entry.getMsg()
    }
}
