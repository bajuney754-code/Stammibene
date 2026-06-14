pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps { git 'https://github.com/bajuney754-code/Stammibene.git' }
        }
        
        stage('Build with Maven') {
            steps { sh 'mvn clean package -DskipTests' }
        }
        
        stage('Deploy to Nexus') {  // <-- your existing mvn deploy stage
            steps { sh 'mvn deploy' }
        }
            stage('Docker Build & Push to Docker Hub') {
    steps {
        script {
            // 1. Get version from pom.xml
            def appVersion = sh(
                script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', 
                returnStdout: true
            ).trim()
            env.IMAGE_TAG = appVersion
            echo "Building image with tag: ${env.IMAGE_TAG}"
        }

        // 2. Build + Push to Docker Hub
        withCredentials([usernamePassword(
            credentialsId: 'docker-registry-creds',  // same cred ID you created in Jenkins
            usernameVariable: 'DOCKER_USER', 
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
                # Login to Docker Hub
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                
                # Build image: docker.io/username/repo:tag
                docker build -t $DOCKER_USER/your-app:$IMAGE_TAG .
                
                # Push image
                docker push $DOCKER_USER/your-app:$IMAGE_TAG
                
                # Tag as latest for release versions only
                if [[ "$IMAGE_TAG" != *SNAPSHOT* ]]; then
                    docker tag $DOCKER_USER/your-app:$IMAGE_TAG $DOCKER_USER/your-app:latest
                    docker push $DOCKER_USER/your-app:latest
                fi
            '''
        }
    }
    post {
        always {
            sh 'docker logout || true'
        }
    }
}
