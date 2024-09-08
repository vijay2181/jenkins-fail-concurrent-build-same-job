# Jenkins Pipeline for Managing Concurrent Builds With Feedback

* created jenkins pipeline script that manages concurrent builds of the same job
* took higher priority on conserving resources and avoiding potential conflicts or race conditions, since we have low resources and high build time.
* when a build is running and another build request comes in for same PR, the new request is failed immediately with an appropriate message that "currenly on-going build is in progress, won't perform this action right away", and it is not queued.
* so for each build, you will get status back as a response in form of comments inside PR 


```
pipeline {
    agent any

    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds() // Ensure only one build per branch at a time
    }

    environment {
        GITHUB_TOKEN = credentials('GitHub-Token') // Add your GitHub token credential
        BUILD_ABORTED_CONCURRENT = 'false'
    }

    stages {
        stage('Check for concurrent builds') {
            steps {
                script {
                    // Get the current branch name
                    def branchName = env.BRANCH_NAME
                    
                    // Get the current job name
                    def jobName = env.JOB_NAME
                    
                    // Get the current build number
                    def buildNumber = env.BUILD_NUMBER.toInteger()
                    
                    // Check for running builds of the same branch
                    def isBuildRunning = false
                    def job = Jenkins.instance.getItemByFullName(jobName)
                    job.builds.each { build ->
                        if (build.isBuilding() && build.number != buildNumber && build.displayName.contains(branchName)) {
                            isBuildRunning = true
                            return false
                        }
                    }
                    
                    // Abort the build if another build is running
                    if (isBuildRunning) {
                        currentBuild.result = 'ABORTED'
                        env.BUILD_ABORTED_CONCURRENT = 'true'
                        error("A build for the branch ${branchName} is already in progress. Aborting this build.")
                    }
                }
            }
        }
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Your build steps here
                    echo 'Building...'
                    // Intentionally fail the build for testing
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Your test steps here
                    echo 'Testing...'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Your deploy steps here
                    echo 'Deploying...'
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean workspace after build
        }

        success {
            script {
                withCredentials([string(credentialsId: 'GitHub-Secret', variable: 'GITHUB_TOKEN')]) {
                    def prComment = "Build succeeded for ${env.BRANCH_NAME}"
                    def prNumber = env.BRANCH_NAME.split('-')[1]
                    def apiUrl = "https://github.com/vijay2181/jenkins-multibranch-pipeline-build-on-PR/issues/${prNumber}/comments"

                    // Ensure the token and URL are used securely
                    sh """
                    curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: token ${GITHUB_TOKEN}" -d '{"body":"${prComment}"}' ${apiUrl}
                    """
                }
            }
        }

        failure {
            script {
                withCredentials([string(credentialsId: 'GitHub-Secret', variable: 'GITHUB_TOKEN')]) {
                    // Determine if the failure was due to a concurrent build or a regular failure
                    def prNumber = env.BRANCH_NAME.split('-')[1] // Extract PR number from branch name, assuming branch name is like PR-16
                    def prComment
                    def apiUrl = "https://github.com/vijay2181/jenkins-multibranch-pipeline-build-on-PR/issues/${prNumber}/comments"
                    if (env.BUILD_ABORTED_CONCURRENT == 'true') {
                        prComment = "Build aborted due to a concurrent build for ${env.BRANCH_NAME}"
                    } else {
                        prComment = "Build failed for ${env.BRANCH_NAME}"
                    }
                    sh """
                    curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: token ${GITHUB_TOKEN}" -d '{"body":"${prComment}"}' ${apiUrl}
                    """
                }
            }
        }
    }
}
```
