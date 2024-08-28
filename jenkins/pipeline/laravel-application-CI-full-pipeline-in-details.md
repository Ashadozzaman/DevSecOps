### Jenkins Pipeline Setup

This guide will walk you through setting up a Jenkins pipeline, configuring build triggers, and creating an agent to connect Jenkins with multiple servers.

---

### Pipeline Build Triggers

Jenkins allows you to set up various build triggers for your pipeline. Two common triggers are:

#### 1. GitHub Hook Trigger for GITScm Polling
- This is the default trigger provided by Jenkins for managing GitHub webhooks.
  
#### 2. Generic Webhook Trigger
- This trigger allows more dynamic management of Git actions.
- **Installation**: Install the `Generic Webhook Trigger` plugin.
- **Webhook URL**: Use the following URL to trigger the webhook:
  ```
  https://cicd.training.bd/generic-webhook-trigger/invoke?token=TOKEN_HERE
  ```
- **Post Content Parameters**: Configure the following parameters:
  - `Variable`: `action`, `Expression`: `$.action`
  - `Variable`: `base`, `Expression`: `$.pull_request.base.ref`
  - `Variable`: `head`, `Expression`: `$.pull_request.base.ref`
  - `Variable`: `success`, `Expression`: `$.pull_request.merged`
- **Optional Filters**:
  - **Expression**: `^closed main dev true$` - This defines the structure to write values for each variable.
  - **Text**: `$action $base $head $success` - Call the post content variables.

---

### Creating a Jenkins Agent

To connect Jenkins with multiple servers, you'll need to create a permanent agent:

1. **Create a Node**:
   - Navigate to `Manage Jenkins` > `Manage Nodes and Clouds` > `New Node`.
   - Set up a `Permanent Agent`.

2. **Agent Configuration**:
   - **Executors**: Set the number of executors (recommended: 2).
   - **Remote Root Directory**: Specify the directory on the agent machine where Jenkins will store files.
   - **Launch Method**: Choose `Launch agent via SSH`.
   - **Host**: Provide the agent's hostname or IP address.
   - **Credentials**: Set up SSH credentials for the agent.
     - For AWS, use the key-pair provided by AWS.
     - For other VMs, connect using SSH keys. Add the public key (`id_ed25519.pub`) from your Jenkins server to the `authorized_keys` file on the agent VM. The private key is used in Jenkins credentials.
     ```
     root@Management2:~# cd .ssh/
     root@Management2:~/.ssh# ls
     authorized_keys  id_ed25519  id_ed25519.pub  id_rsa  id_rsa.pub  known_hosts
     ```

---

### Jenkins Pipeline Script

Below is an example of a Jenkins pipeline script. This script demonstrates how to clean up the workspace, check out a Git repository, manage environment variables, restore backups, and run project builds.

```groovy
pipeline {
    agent {
        label 'Live-Template'
    }

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_REPO = "https://github.com/Ashadozzaman/Test-Builder.git"
        CLIENT_APP_NAME = "Test-Builder"
        CONFIG_PROJECT_NAME = "test_builder"
        ENV_FILE_PATH = "/var/www/html/env_files/.env" // Stored outside the workspace
        CUSTOM_WORKSPACE = "/var/www/html/template-builder"
    }

    options {
        skipDefaultCheckout(true) // Skips the default checkout to use our custom directory
    }

    stages {
        stage('CLEANUP WORKSPACE') {
            steps {
                script {
                    echo "Cleaning up workspace..."
                    sh "sudo rm -rf /var/www/html/caches /var/www/html/remoting /var/www/html/remoting.jar ${CUSTOM_WORKSPACE}"
                }
            }
        }

        stage("CHECKOUT GIT REPO") {
            steps {
                dir("${CUSTOM_WORKSPACE}") {
                    checkout([$class: 'GitSCM',
                              branches: [[name: 'main']],
                              userRemoteConfigs: [[url: "${GIT_REPO}", credentialsId: 'Git-Hub-Credential-Template-Ashadozzaman']]
                    ])
                }
            }
        }

        stage("VERIFY CHECKOUT") {
            steps {
                script {
                    sh "ls -la ${CUSTOM_WORKSPACE}"
                }
            }
        }

        stage('Symlink .env') {
            steps {
                script {
                    sh "ln -sf ${ENV_FILE_PATH} ${CUSTOM_WORKSPACE}/.env"
                }
            }
        }

        stage('Restore & Permission') {
            steps {
                echo 'Restoring Backup of env and adjusting permissions'
                sh "chown -R www-data:www-data ${CUSTOM_WORKSPACE}/"
                sh "chmod -R 777 ${CUSTOM_WORKSPACE}/"

                // Set permissions for specific directories
                sh "sudo chown -R www-data:www-data ${CUSTOM_WORKSPACE}/storage ${CUSTOM_WORKSPACE}/bootstrap/cache"
                sh "sudo chmod -R 775 ${CUSTOM_WORKSPACE}/storage ${CUSTOM_WORKSPACE}/bootstrap/cache"
            }
        }
        
        stage('Install Dependencies') {
            steps {
                dir("${CUSTOM_WORKSPACE}") {
                    sh 'npm install' // Install npm dependencies
                }
            }
        }
        
        stage('Install Vite') {
            steps {
                dir("${CUSTOM_WORKSPACE}") {
                    sh 'npm install vite --save-dev' // Install Vite as a development dependency
                }
            }
        }
        
        stage('Build Project') {
            steps {
                dir("${CUSTOM_WORKSPACE}") {
                    sh 'npm run build' // Build the project
                }
            }
        }
        
        stage('Update Dependencies') {
            steps {
                dir("${CUSTOM_WORKSPACE}") {
                    sh 'composer update' // Install Composer dependencies
                }
            }
        }

        stage('Database Migration') {
            steps {
                dir("${CUSTOM_WORKSPACE}") {
                    // Perform database-related tasks
                    // sh 'php artisan migrate --force' // Uncomment if you need to run migrations
                    sh 'php artisan cache:clear'
                    sh 'php artisan config:clear'
                    sh 'php artisan optimize:clear'
                }
            }
        }

        stage('Verify Directory After Checkout') {
            steps {
                script {
                    sh 'ls -la /var/www/html'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed. Cleaning up workspace..."
        }
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline execution failed.'
        }
    }
}
```

---

### Key Changes:
1. **Pipeline Build Triggers**: Organized into clear subsections for better readability.
2. **Agent Creation**: Expanded and clarified steps for connecting Jenkins to multiple servers.
3. **Pipeline Script**: Rearranged some of the stages and added comments for clarity. Combined similar cleanup steps into one stage.
