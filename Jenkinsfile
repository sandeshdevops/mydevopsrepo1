
pipeline {
    agent any

    environment {
        registryCredentials = 'dockerhub_credentials'
        dockerImage = 'sandeshyashlaha/devopsproject2'
        dockerTag = 'latest'
        dockerfilePath = 'docker/creating-image/Dockerfile'  // Assuming Dockerfile is at the root of the repository
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from GitHub
                git branch: 'main', url: 'https://github.com/sandeshdevops/mydevopsrepo1.git'
            }
        }

        stage('Build') {
            steps {
                // Build Docker image
                script {
                    dockerImage = docker.build("${dockerImage}:${dockerTag}", "-f ${dockerfilePath} .")
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    dockerImage.inside('-p 5001:5000') {  // Map port 5001 on the host to port 5000 in the container
                        sh 'python3 docker/creating-image/app.py --host=0.0.0.0 &'  // Bind to all IP addresses
                        sleep 10  // Wait for the Flask app to start
                    }
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', registryCredentials) {
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! Docker image built, tested, and pushed successfully.'
        }
        failure {
            echo 'Pipeline failed! Check the Jenkins console output for details.'
        }
    }
}
