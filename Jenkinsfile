pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                scmSkip(deleteBuild: true, skipPattern:'.*\\[ci skip\\].*')
            // Run build steps only when changes are detected in the domains folder or a new branch is pushed
            git url: 'https://github.com/AvinashNagula/testskipci.git',
                    branch: 'main'
            // Add your build steps here, /e.g.:
            // sh 'mvn clean package'
            }
        }
        stage('test') {
            steps {
                scmSkip(deleteBuild: true, skipPattern:'.*\\[ci skip\\].*')
                echo "hello...."
            }
        }
    }
}