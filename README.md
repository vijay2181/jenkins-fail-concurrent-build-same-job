# Jenkins Pipeline for Managing Concurrent Builds

## Introduction

This README explains the implementation of a Jenkins pipeline script that manages concurrent builds of the same job. The goal is to prevent multiple instances of the same job from running simultaneously, thus conserving resources and avoiding potential conflicts or race conditions. This script fails the current build if another instance of the same job is already running, ensuring that resources are managed efficiently.

## Implementation

### Pipeline Script

The following pipeline script checks for concurrent builds of the same job. If any are detected, it fails the current build to avoid resource conflicts.

```groovy
pipeline {
    agent any
    stages {
        stage('Check for concurrent builds') {
            steps {
                script {
                    def currentBuildNumber = currentBuild.number
                    def concurrentBuilds = 0

                    for (build in currentBuild.rawBuild.parent.builds) {
                        if (build.isBuilding() && build.number != currentBuildNumber) {
                            concurrentBuilds++
                        }
                    }

                    if (concurrentBuilds > 0) {
                        error "Already same Job is Running. In order to manage resources, failing this build and not keeping it in queue. Please try after the existing build is completed: ${concurrentBuilds}"
                    }
                }
            }
        }
        stage ('Sleep') {
            steps {
                sh 'sleep 120'
            }
        }
    }
    post {
        always {
            script {
                echo 'Post build actions'
                // Any post build actions
            }
        }
    }
}
```

```
- for above script inorder to work, you need to approve script multiple times
- you will get below error, follow the link and approve the script.
- Scripts not permitted to use method hudson.model.Run getNumber. Administrators can decide whether to approve or reject this signature.

- below are approval stages

method hudson.model.Job getBuilds
method hudson.model.Run getNumber
method hudson.model.Run getParent
method hudson.model.Run isBuilding
method jenkins.model.Jenkins getItemByFullName java.lang.String
method org.jenkinsci.plugins.workflow.support.steps.build.RunWrapper getRawBuild
staticMethod jenkins.model.Jenkins getInstance
```


### Breakdown of the Script

#### Check for Concurrent Builds

1. **Stage: Check for concurrent builds**
    - **Purpose**: To detect if there are any other running builds of the same job.
    - **Steps**:
        - Retrieve the current build number using `currentBuild.number`.
        - Initialize a counter `concurrentBuilds` to zero.
        - Iterate through all builds of the current job using `currentBuild.rawBuild.parent.builds`.
        - If a build is running (`isBuilding()`) and its build number is different from the current build, increment the `concurrentBuilds` counter.
        - If `concurrentBuilds` is greater than zero, fail the build with an appropriate error message.

#### Example Sleep Stage

2. **Stage: Sleep**
    - **Purpose**: To simulate a build step. This can be replaced with actual build steps as required.
    - **Steps**:
        - Sleep for 120 seconds using `sh 'sleep 120'`.

#### Post Build Actions

3. **Post Build Actions**
    - **Purpose**: To execute any actions that need to run after the build, regardless of the build result (success, failure, etc.).
    - **Steps**:
        - Print a message indicating the execution of post-build actions.

## Why This Was Implemented

### Resource Management

Running multiple instances of the same job simultaneously can lead to resource contention, where builds compete for CPU, memory, disk I/O, and other resources. This can slow down the builds and affect the performance of other jobs running on the same Jenkins instance. By preventing concurrent builds, we ensure that resources are used efficiently and builds run faster.

### Avoiding Conflicts

Concurrent builds of the same job can lead to conflicts, such as:
- **File Access Conflicts**: Multiple builds trying to read/write the same file simultaneously can cause corruption or inconsistencies.
- **Database Conflicts**: Simultaneous access to the same database can lead to deadlocks or data corruption.
- **Dependency Conflicts**: Builds may have dependencies that cannot be used concurrently, leading to failures.

By failing the current build when another instance is running, we avoid these conflicts and ensure that each build runs in a clean, isolated environment.

## Example Use Cases

### Continuous Integration

In a continuous integration environment, builds are triggered frequently (e.g., on every commit). If multiple commits are pushed in quick succession, it can trigger multiple builds of the same job. This script ensures that only one build runs at a time, preventing resource contention and conflicts.

### Scheduled Jobs

For scheduled jobs that run periodically (e.g., nightly builds), there is a risk that a build might take longer than expected, overlapping with the next scheduled build. This script ensures that the next build does not start until the previous one completes, maintaining a clear separation between builds.


## Differences Between Queuing and Failing Concurrent Builds

When managing Jenkins jobs, there are generally two approaches to handling concurrent builds:

1. **Queuing Builds**: When a build is running and another build request comes in, the new request is placed in a queue and will start automatically once the current build finishes.
2. **Failing Concurrent Builds**: When a build is running and another build request comes in, the new request is failed immediately with an appropriate message, and it is not queued.


#### Queuing Builds

1. **Automatic Execution**: The queued build starts automatically once the current build completes.
2. **Build Order**: The order of build requests is preserved, and all builds will eventually run, albeit with a delay.
3. **Resource Contention**: Depending on the build frequency, a long queue can form, leading to potential resource contention and delays in build processing.
4. **Resource Management**: The queued build can create resource bottlenecks if multiple builds are queued up, as each build waits for resources to become available.

#### Failing Concurrent Builds

1. **Immediate Feedback**: The concurrent build request fails immediately, providing immediate feedback to the user or system that triggered the build.
2. **Manual Intervention**: The user or system must manually re-trigger the build once the current build completes, ensuring that builds are explicitly controlled.
3. **Resource Efficiency**: By failing the build instead of queuing it, resources are not tied up with queued builds, leading to more efficient resource utilization.
4. **Reduced Wait Times**: Since builds are not queued, there are no extended wait times for queued builds to start, which can be crucial in environments where quick feedback is essential.

### Why Implement Failing Concurrent Builds?

1. **Resource Management**: In environments with limited resources, failing concurrent builds can prevent resource bottlenecks and ensure that resources are available for other critical tasks.
2. **Avoiding Long Queues**: Long build queues can delay the feedback cycle, especially in continuous integration environments where quick feedback is crucial. Failing concurrent builds avoids long queues.
3. **Immediate Awareness**: When a build fails due to concurrency, it brings immediate attention to the fact that a build is already running, prompting users to investigate and manage build triggers more effectively.
4. **Controlled Build Triggers**: Failing concurrent builds forces users to manage build triggers manually, ensuring that builds are only triggered when necessary and avoiding redundant or unnecessary builds.
5. **Custom Handling**: It allows for custom handling of build failures due to concurrency, such as sending notifications, logging specific messages, or integrating with other systems to handle build failures appropriately.

### Example Use Case

Consider a scenario where you have a Jenkins job that performs a critical deployment to a production environment. Deployments should not overlap, as this could lead to inconsistencies or downtime. By failing concurrent builds:

- **Immediate Notification**: The team is immediately notified that another deployment is in progress.
- **Controlled Deployments**: The team can manually assess the situation and re-trigger the deployment once the current one completes, ensuring controlled and orderly deployments.
- **Avoiding Downtime**: This approach reduces the risk of downtime or deployment conflicts, maintaining the stability of the production environment.

### Conclusion

While queuing builds is useful in many scenarios, failing concurrent builds provides greater control, immediate feedback, and more efficient resource management, especially in environments where resources are limited, and quick feedback is essential. This approach helps maintain the stability and reliability of the CI/CD pipeline by avoiding potential conflicts and resource contention.

- Failing concurrent builds provides greater control, immediate feedback, and efficient resource management, especially in environments requiring quick feedback and stability. T
- his approach helps maintain the reliability of the CI/CD pipeline by avoiding conflicts and resource contention.


## How It Is Useful

### Efficiency

- **Resource Utilization**: By preventing concurrent builds, resources are used more efficiently. Each build gets the necessary resources without having to compete with other builds.
- **Build Performance**: With fewer builds running concurrently, each build can run faster and more reliably.

### Reliability

- **Consistency**: Ensuring that only one build runs at a time avoids conflicts, resulting in more consistent and reliable builds.
- **Error Prevention**: By failing the build early if another build is running, we prevent potential errors that could arise from concurrent execution.

### Simplified Management

- **Queue Management**: Instead of queuing multiple builds, this script fails the current build, allowing the user to trigger it manually when the previous build completes. This simplifies build management and ensures that resources are not unnecessarily tied up.

## Conclusion

This Jenkins pipeline script provides an effective way to manage concurrent builds, ensuring efficient resource utilization and reliable builds. By implementing this script, you can prevent conflicts, avoid resource contention, and maintain a more stable CI/CD environment.

---

Feel free to modify the content as per your specific requirements or add more details if necessary.
