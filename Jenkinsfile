@Library("Git-Libraries") _

pipeline { 
    agent {label "Agent1"}

    stages {
        stage('Code') {
            
            steps {
                script{
                clone("https://github.com/IkramAnsari01/DjangoReactApp","main")
                }
                
            }
        }
        
        stage('Build') {
            steps {
                script{
                buildImage("notes-app","latest")
                }
            }
        }
        stage('DockerHub Push'){
            steps{
                
                echo ' Pushing build Image to Docker Hub'
                withCredentials([usernamePassword(
                        'credentialsId':"DockerHubCred",
                        passwordVariable: "DocPass",
                        usernameVariable: "DocUser")]){
                            script{
                                dockerPush("DocUser", "DocUser", "notes-app","latest")
                            }
                         }
            }
        }
        
        stage('Test') {
            steps {
                echo 'Testing the code'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying the code'
                sh '''
                    # 1. Stop AND Remove the old container
                    echo "Stopping old container..."
                    docker stop notes-app || true
                    
                    echo "Removing old container..."
                    docker rm notes-app || true   # <--- THIS WAS MISSING
                    
                    # 2. Force kill port 8000 (Safety measure)
                    echo "Freeing port 8000..."
                    fuser -k 8000/tcp || true
                    
                    # 3. Build the new image
                    echo "Building new image..."
                    docker build --no-cache -t notes-app:latest .
                    
                    # 4. Run the new container
                    echo "Starting new container..."
                    docker run -d --name notes-app -p 8000:8000 notes-app:latest
                    
                    # 5. Cleanup and Verification
                    echo "Removing dangling images..."
                    docker image prune -f
                    
                    echo "Verifying deployment..."
                    sleep 5
                    docker ps | grep notes-app
                '''
            }
        }
    }
}
