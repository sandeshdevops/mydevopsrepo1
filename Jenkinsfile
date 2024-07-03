pipeline {
    agent any

    environment {
        registryCredentials = '271bdd6e-0a51-49bb-9140-bf91dcea0235'
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

        stage('Approval') {
            steps {
                script {
                    input message: 'Approve deployment?', ok: 'Deploy'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Stop and remove existing container if it exists
                    sh """
                    if [ \$(docker ps -a -q -f name=project2) ]; then
                        docker stop project2
                        docker rm project2
                    fi
                    """

                    // Run Docker container
                    sh "docker run -d -p 8080:80 --name project2 ${dockerImage}:${dockerTag}"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! Docker image built, tested, pushed, and deployed successfully.'
        }
        failure {
            echo 'Pipeline failed! Check the Jenkins console output for details.'
        }
    }
}
