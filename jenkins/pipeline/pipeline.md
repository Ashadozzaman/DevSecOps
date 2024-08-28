### Create Pipeline Job In Jenkins
- We can use two type `Build Triggers`

**GitHub hook trigger for GITScm polling:**
- Default Manage git weebhook trigger

**Generic Webhook Trigger:**
- Using this dynamiclly manage git action
- Install `generic webhook triger plugin` 
- https://cicd.training.mygov.bd/generic-webhook-trigger/invoke?token=TOKEN_HERE
- **Post content parameters**
    ```
    Set `Variable` "action", `Expression` "$.action"
    Set `Variable` "base", `Expression` "$.pull_request.base.ref"
    Set `Variable` "head", `Expression` "$.pull_request.base.ref"
    Set `Variable` "success", `Expression` "$.pull_request.merged"
    ```
- **Post content parameters**
    - `Optional filter:` Expression ->  `^closed main dev true$ `// This is the structure to write value for each variable
    - `Optional filter:` Text ->  `$action $base $head $success` // Post Content variable call this away

### Create Agent 
- Through by agent connect jenkins server with multiple server.
- First Create `Node`- Permarent Agent
- Executer 2
- Remote Root dir
- Lunch Via SSH
- Agent Host
- Set Key- 
    - For AWS set key-pair
    - Connect Other VM with jenkins server
        ```
        root@Management2:~# cd .ssh/
        root@Management2:~/.ssh# ls
        authorized_keys  id_ed25519  id_ed25519.pub  id_rsa  id_rsa.pub  known_hosts
        ```
        - This public key (`id_ed25519`) add in agent VM authorized_keys & private key(`id_ed25519.pub`) need to agent credential.

```
pipeline {
    agent {
        label 'Live-Template'
    }

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_REPO = "https://github.com/Ashadozzaman/Template-Builder.git"
        CLIENT_APP_NAME = "Template-Builder"
        CONFIG_PROJECT_NAME = "template_builder"
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
                    sh "rm -rf ${CUSTOM_WORKSPACE}"
                }
            }
        }
        
        stage("Cleanup Workspace") {
            steps {
                script {
                    echo "Cleaning up workspace..."
                    sh 'sudo rm -rf /var/www/html/caches /var/www/html/remoting /var/www/html/remoting.jar ${CUSTOM_WORKSPACE}'
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
                    // Install npm dependencies
                    sh 'npm install'
                }
            }
        }
        stage('Install Vite') {
            steps {
                dir("${CUSTOM_WORKSPACE}") {
                    // Install Vite as a development dependency
                    sh 'npm install vite --save-dev'
                }
            }
        }
        stage('Build Project') {
            steps {
                dir("${CUSTOM_WORKSPACE}") {
                    // Build the project
                    sh 'npm run build'
                }
            }
        }
        
        stage('Update Dependencies') {
            steps {
                dir("${CUSTOM_WORKSPACE}") {
                    // Install Composer dependencies
                    sh 'composer update'
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
            echo "Cleaning up workspace..."
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