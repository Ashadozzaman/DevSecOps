## Jenkins pipeline with post actions

In Jenkins pipelines, you can use the `post` block to define actions that should be taken after the pipeline's execution, regardless of the pipeline's success or failure. The `post` block can contain multiple conditions such as `always`, `success`, `failure`, `unstable`, `changed`, etc.

Here's an example of a Jenkins pipeline with `post` actions:

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                // Add your build steps here
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // Add your test steps here
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Add your deploy steps here
            }
        }
    }
    
    post {
        always {
            echo 'This always runs'
            // Steps that should run always, like cleanup
        }
        success {
            echo 'This runs only if the build succeeds'
            // Steps that should run only if the build succeeds
        }
        failure {
            echo 'This runs only if the build fails'
            // Steps that should run only if the build fails
        }
        unstable {
            echo 'This runs if the build is unstable'
            // Steps that should run if the build is unstable
        }
        changed {
            echo 'This runs if the result of the build has changed'
            // Steps that should run if the build result has changed
        }
    }
}
```

### Explanation:

1. **pipeline**: Defines the entire Jenkins pipeline.
2. **agent any**: Specifies that the pipeline can run on any available agent.
3. **stages**: Defines the different stages of the pipeline (`Build`, `Test`, `Deploy`).
4. **post**: Defines the actions to be taken after the stages have executed.
   - **always**: Actions that run regardless of the pipeline outcome.
   - **success**: Actions that run only if the pipeline succeeds.
   - **failure**: Actions that run only if the pipeline fails.
   - **unstable**: Actions that run if the pipeline is unstable (e.g., test failures but build succeeds).
   - **changed**: Actions that run if the build result has changed compared to the previous run.

You can add any necessary steps within these blocks, such as sending notifications, cleaning up resources, archiving artifacts, or anything else needed after the pipeline execution.