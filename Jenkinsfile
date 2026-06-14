pipeline {
    // This tells Jenkins to run this build on any available worker machine
    agent any

    tools {
	dockerTool 'myDocker'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Pulls down your source code containing the pom.xml and Dockerfile
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
                    
                    // Save it directly to the env object so the shell block below can read it
                    env.IMAGE_TAG = appVersion
                    echo "Building image with tag: ${env.IMAGE_TAG}"
                }

                // 2. Build + Push using Jenkins credentials
                withCredentials([usernamePassword(
                    credentialsId: 'docker-registry-creds', // Must match the ID created in your Jenkins UI
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    // Triple double-quotes allow Jenkins to inject the $IMAGE_TAG variable

		    withEnv(["PATH+DOCKER=${tool 'myDocker'}/bin"]){
                    sh """
                        # Login to Docker Hub (omitting the domain defaults to docker.io)
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        
                        # Build image (Replace 'your-app' with your actual Docker Hub repository name)
                        docker build -t \$DOCKER_USER/blabla:\$IMAGE_TAG .
                        
                        # Push image
                        docker push \$DOCKER_USER/blabla:\$IMAGE_TAG
                        
                        # Tag as latest for production/stable releases only
                        if [[ "\$IMAGE_TAG" != *SNAPSHOT* ]]; then
                            docker tag \$DOCKER_USER/blabla:\$IMAGE_TAG \$DOCKER_USER/your-app:latest
                            docker push \$DOCKER_USER/blabla:latest
                        fi
                    """
		   }
                }
            }
            post {
                always {
		     withEnv(["PATH+DOCKER=${tool 'myDocker'}/bin"]) {
                    // Clears credentials from the local Jenkins worker engine memory
                    sh 'docker logout || true'
                    
                    // Optional: Cleans up local images to save server disk space
                    sh "docker rmi \$DOCKER_USER/blabla:\$IMAGE_TAG || true"
                 }
             }
         }
      }
  }
}
