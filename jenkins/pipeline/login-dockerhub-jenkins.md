## Docker Hub Login & Push Image

### Login By Default Credential

### Login with secure awayed to configure here
Dashboard>Manage Jenkins>Credentials>System>Global credentials (unrestricted)

### Reference Credentials in the Pipeline Script
```
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credential', passwordVariable: 'passd', usernameVariable: 'uname')]) {
                    sh 'echo ${passd} | docker login --username ${uname} --password-stdin'
                    // Push Docker image to DockerHub
                    sh 'docker push ${DOCKER_USERNAME}/frontend:latest'
                    // Logout from DockerHub
                    sh 'docker logout'
                }
            }
        }
    }
}
```

### In details Flows

```
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub_credential'  // Replace with your actual credentials ID
    }

    stages {
        stage('Hello') {
            steps {
                script {
                    echo 'Hello World'
                    // Checkout code from GitHub
                    git branch: 'main', url: 'https://github.com/GitUserNameHere/ci_cd_demo-project.git'
                    
                    // Build Docker image
                    sh 'docker build -t dockerUserName/frontend:latest .'
                    
                    // Login to DockerHub using Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo ${DOCKER_PASSWORD} | docker login --username ${DOCKER_USERNAME} --password-stdin'
                        
                        // Push Docker image to DockerHub
                        sh 'docker push ${DOCKER_USERNAME}/frontend:latest'
                        
                        // Logout from DockerHub
                        sh 'docker logout'
                    }
                }
            }
        }
    }
}
```
