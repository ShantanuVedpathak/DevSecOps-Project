pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shantanuvedpathak01/netflix-devsecops:latest"
        CONTAINER_NAME = "netflix-devsecops"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                credentialsId: 'github-creds',
                url: 'https://github.com/ShantanuVedpathak/DevSecOps-Project.git'
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage("Docker Build image"){
            steps{
                script{   
                       sh "docker build --build-arg TMDB_V3_API_KEY=a7e0428cc5070744019a0ed61a4d8abf -t netflix-devsecops ."
                       sh "docker tag netflix shantanuvedpathak01/netflix-devsecops:latest "
                      
                    }
                }
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh 'trivy image $DOCKER_IMAGE'
            }
        }

        stage('Push Docker Image') {
            steps {

                withCredentials([
                  usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $DOCKER_IMAGE --disable-content-trust
                    '''
                }
            }
        }

        stage('Deploy Container Locally (Optional Test)') {
            steps {

                sh '''
                docker rm -f $CONTAINER_NAME || true

                docker run -d \
                --name $CONTAINER_NAME \
                -p 8081:80 \
                $DOCKER_IMAGE
                '''
            }
        }
    }

    post {

        success {
            echo 'Pipeline executed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }

        always {
            sh 'docker system prune -f || true'
        }
    }
}