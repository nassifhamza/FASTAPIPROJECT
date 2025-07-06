// Jenkinsfile for FastAPI Full Stack Template
pipeline {
    agent any

    environment {
        // Define environment variables for your services
        // These should match the credentials in your docker-compose.yml or be managed by Jenkins secrets
        POSTGRES_DB = 'mydatabase'
        POSTGRES_USER = 'myuser'
        POSTGRES_PASSWORD = 'mypassword'
        // For SonarQube, you'd add similar variables
        // SONAR_TOKEN = credentials('sonarqube-token') // Example for Jenkins credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/nassifhamza/FASTAPIPROJECT.git'
            }
        }

        stage('Build Backend' ) {
            steps {
                script {
                    // Navigate to the backend directory
                    dir('backend') {
                        // Build the Docker image for the backend
                        // Ensure Docker is accessible from Jenkins agent (privileged mode in docker-compose)
                        sh 'docker build -t fastapi-backend .'
                    }
                }
            }
        }

        stage('Run Backend Tests') {
            steps {
                script {
                    dir('backend') {
                        // Run tests using the Docker image or directly if dependencies are installed on agent
                        // For this example, we'll assume a simple test execution within the container
                        sh 'docker run --rm -v $(pwd):/app -w /app fastapi-backend bash -c "uv sync && bash scripts/test.sh"'
                    }
                }
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    // Ensure SonarQube Scanner is installed in Jenkins
                    // Configure SonarQube server in Jenkins -> Manage Jenkins -> Configure System
                    // Add SonarQube Scanner as a tool in Jenkins -> Manage Jenkins -> Global Tool Configuration
                    withSonarQubeEnv(credentialsId: 'sonarqube-token', installationName: 'SonarQube') {
                        sh 'sonar-scanner \
                          -Dsonar.projectKey=fastapi-project \
                          -Dsonar.sources=backend/app \
                          -Dsonar.host.url=http://sonarqube:9000 \
                          -Dsonar.login=${SONAR_TOKEN}'
                    }
                }
            }
        }

        // Optional: Build Frontend (if applicable )
        // stage('Build Frontend') {
        //     steps {
        //         script {
        //             dir('frontend') {
        //                 sh 'npm install'
        //                 sh 'npm run build'
        //             }
        //         }
        //     }
        // }

        stage("Push to Nexus") {
            steps {
                script {
                    // Use withCredentials to securely access Nexus credentials
                    withCredentials([usernamePassword(credentialsId: 'nexus-user', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                        sh 'docker tag fastapi-backend nexus.devops.local:8081/fastapi-backend:latest'
                        sh "docker login -u ${NEXUS_USERNAME} -p ${NEXUS_PASSWORD} nexus.devops.local:8081"
                        sh 'docker push nexus.devops.local:8081/fastapi-backend:latest'
                    }
                }
            }
        }

        // Optional: Deployment Stage
        // This stage would depend on your deployment strategy (e.g., deploying to a Kubernetes cluster, VM, etc.)
        // stage('Deploy') {
        //     steps {
        //         echo 'Deploying application...'
        //     }
        // }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
