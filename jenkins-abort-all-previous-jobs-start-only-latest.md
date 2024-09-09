
# jenkins-abort-all-previous-jobs-start-only-latest

```
Author:  vijay
This script works with workflow to cancel other running builds for the same job
Use case: many build may go to QA, but only the build that is accepted is needed,
the other builds in the workflow should be aborted
- implemented in multi-branch pipeline
```


```
pipeline {
    agent any
    
    options { 
        buildDiscarder(logRotator(numToKeepStr: '5')) 
    }
    environment {
        GITHUB_TOKEN = credentials('GitHub-Token')
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Cancel Previous Builds') {   
            steps {
                script {
                    cancelPreviousBuilds()
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Running build stage...'
                sh 'sleep 20'
                // Add your build steps here
            }
        }

        stage('Test') {
            steps {
                echo 'Running test stage...'
                // Add your test steps here
            }
        }

        stage('Deploy') {
            steps {
                echo 'Running deploy stage...'
                // Add your deploy steps here
            }
        }
    }

    post {
        always {
            echo 'This will always run'
        }
        failure {
            echo 'This will run only if failed'
        }
    }
}



def cancelPreviousBuilds() {
 // Check for other instances of this particular build, cancel any that are older than the current one
 def jobName = env.JOB_NAME
 def currentBuildNumber = env.BUILD_NUMBER.toInteger()
 def currentJob = Jenkins.instance.getItemByFullName(jobName)

 // Loop through all instances of this particular job/branch
 for (def build : currentJob.builds) {
 if (build.isBuilding() && (build.number.toInteger() < currentBuildNumber)) {
 echo "Older build still queued. Sending kill signal to build number: ${build.number}"
 build.doStop()
 }
 }
}
```
  
