pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // 1. Get version from pom.xml so tag matches Maven artifact
                    def appVersion = sh(
                        script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', 
                        returnStdout: true
                    ).trim()
                    
                    echo "Building image version: ${appVersion}"
                    
                    // 2. Use the native Jenkins Docker DSL plugin to build your Dockerfile
                    // Replace 'your-app' with your actual Docker Hub repository name
                    def dockerImage = docker.build("your-app:${appVersion}", ".")

                    // 3. Connect to Docker Hub registry securely and push the layers
                    // 'docker-registry-creds' must match the exact ID created in your Jenkins UI
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-registry-creds') {
                        
                        // This handles the login and pushing automatically!
                        dockerImage.push()
                        
                        // Tag and push as 'latest' for non-SNAPSHOT versions
                        if (!appVersion.contains("SNAPSHOT")) {
                            dockerImage.push('latest')
                        }
                    }
                }
            }
            post {
                always {
                    script {
                        echo "Cleaning workspace footprint..."
                        // Local cleanup script
                    }
                }
            }
        }
    }
}

