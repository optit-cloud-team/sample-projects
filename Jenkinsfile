pipeline {
    agent any 
    stages { 
        stage('Git Checkout') { 
            steps { 
                script { 
                    git branch: branch_or_tag, 
                        credentialsId: 'git-PAT', // Provide your credentials ID
                        url: 'https://github.com/optit-cloud-team/sample-projects.git' // Provide your Git repository URL
                }
            } 
        }

        stage('Build with Maven') {
            steps {
                script {
                      sh "${tool 'Maven'} clean install"
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // Assuming your Dockerfile is in the root directory of your repository
                    docker.build("java-maven", ".") // Replace "your-docker-image-name" with your desired image name
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', // Adjust the path and file pattern to match your Maven build output
                                fingerprint: true
            }
        }

        stage('Docker Publish') {
            steps {
                script {
                    // Docker login using credentials
                    withCredentials([usernamePassword(credentialsId: 'bkdockerid', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        
                        def buildId = env.BUILD_ID
                        def imageNameWithTag = "srikanth6520/java-maven:$buildId"
                        sh "docker tag java-maven $imageNameWithTag"
                        sh "docker push $imageNameWithTag"

                        def imageNameWithLatest = "srikanth6520/java-maven:latest"
                        sh "docker tag optit-lab-service $imageNameWithLatest"
                        sh "docker push $imageNameWithLatest"
                    }
                }
            }
        }
    }
}
