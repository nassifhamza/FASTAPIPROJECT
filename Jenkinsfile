// Jenkinsfile for FastAPI Full Stack Template with BuildKit fix
pipeline {
    agent any

    environment {
        // Define environment variables for your services
        POSTGRES_DB = 'mydatabase'
        POSTGRES_USER = 'myuser'
        POSTGRES_PASSWORD = 'mypassword'
        // Enable BuildKit globally for all Docker commands
        DOCKER_BUILDKIT = "1"
        // Nexus repository URL (configurable)
        NEXUS_REGISTRY = "nexus.devops.local:8081"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/nassifhamza/FASTAPIPROJECT'
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    dir('backend') {
                        // Build with BuildKit explicitly enabled
                        sh 'docker build -t fastapi-backend .'
                    }
                }
            }
        }

        stage('Run Backend Tests') {
            steps {
                script {
                    dir('backend') {
                        // Run tests in Docker container
                        sh 'docker run --rm -v $(pwd):/app -w /app fastapi-backend bash -c "uv sync && bash scripts/test.sh"'
                    }
                }
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    // Use Jenkins-configured SonarQube server
                    withSonarQubeEnv(credentialsId: 'sonarqube-token', installationName: 'SonarQube') {
                        sh '''
                            sonar-scanner \
                            -Dsonar.projectKey=fastapi-project \
                            -Dsonar.sources=backend/app \
                            -Dsonar.host.url=http://sonarqube:9000 \
                            -Dsonar.login=${SONAR_TOKEN}
                        '''
                    }
                }
            }
        }

        stage("Push to Nexus") {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'nexus-user',
                            usernameVariable: 'NEXUS_USERNAME',
                            passwordVariable: 'NEXUS_PASSWORD'
                        )
                    ]) {
                        sh """
                            docker tag fastapi-backend ${NEXUS_REGISTRY}/fastapi-backend:latest
                            docker login -u ${NEXUS_USERNAME} -p ${NEXUS_PASSWORD} ${NEXUS_REGISTRY}
                            docker push ${NEXUS_REGISTRY}/fastapi-backend:latest
                        """
                    }
                }
            }
        }

        // Optional: Deployment Stage
        // stage('Deploy') {
        //     steps {
        //         echo 'Deploying application...'
        //         // Add your deployment commands here
        //     }
        // }
    }

    post {
        always {
            echo 'Pipeline finished.'
            // Clean up Docker images to save disk space
            sh 'docker rmi fastapi-backend || true'
            sh "docker rmi ${NEXUS_REGISTRY}/fastapi-backend:latest || true"
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
            // Optional: Send notification on failure
        }
    }
}
