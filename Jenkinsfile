// Jenkinsfile for FastAPI Full Stack Template with Buildx fix
pipeline {
    agent any

    environment {
        // Define environment variables
        POSTGRES_DB = 'mydatabase'
        POSTGRES_USER = 'myuser'
        POSTGRES_PASSWORD = 'mypassword'
        NEXUS_REGISTRY = "nexus.devops.local:8081"
        
        // Disable BuildKit since buildx is missing
        DOCKER_BUILDKIT = "0"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/nassifhamza/FASTAPIPROJECT'
            }
        }

        stage('Install Buildx') {
            steps {
                script {
                    // Install buildx plugin
                    sh '''
                        mkdir -p ~/.docker/cli-plugins
                        curl -sSL https://github.com/docker/buildx/releases/download/v0.13.0/buildx-v0.13.0.linux-amd64 -o ~/.docker/cli-plugins/docker-buildx
                        chmod +x ~/.docker/cli-plugins/docker-buildx
                        docker buildx version
                    '''
                }
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    dir('backend') {
                        // Re-enable BuildKit now that buildx is installed
                        sh '''
                            DOCKER_BUILDKIT=1 docker build -t fastapi-backend .
                        '''
                    }
                }
            }
        }

        stage('Run Backend Tests') {
            steps {
                script {
                    dir('backend') {
                        sh 'docker run --rm -v $(pwd):/app -w /app fastapi-backend bash -c "uv sync && bash scripts/test.sh"'
                    }
                }
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
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
    }

    post {
        always {
            echo 'Pipeline finished.'
            // Clean up Docker images
            sh 'docker rmi fastapi-backend || true'
            sh "docker rmi ${NEXUS_REGISTRY}/fastapi-backend:latest || true"
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
            // Send notification on failure
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME}' (${env.BUILD_NUMBER})",
                body: "Check console output at ${env.BUILD_URL}",
                to: 'hamza@example.com'
            )
        }
    }
}
