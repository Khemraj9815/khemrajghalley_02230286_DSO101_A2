pipeline {
    agent any

    tools {
        // Must match the 'Name' you gave Node in Global Tool Configuration
        nodejs 'Node-20' 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
                // Only runs if a "build" script exists in your package.json
                sh 'npm run build --if-present' 
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
            post {
                always {
                    // Collects test reports for the Jenkins UI
                    junit '**/junit.xml' 
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    try {
                        echo 'Building and pushing Docker image...'
                        def app = docker.build("khemraj9815/node-app:${env.BUILD_ID}")
                        
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-creds') {
                            app.push()
                            app.push('latest')
                        }
                        echo 'Docker image successfully pushed to Docker Hub!'
                    } catch (Exception e) {
                        echo "Docker deployment failed or skipped: ${e.message}"
                        echo 'Continuing pipeline...'
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed! Check the logs.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}
