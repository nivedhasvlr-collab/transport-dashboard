pipeline {
    agent any
    
    parameters {
        string(name: 'DOCKER_IMAGE', defaultValue: 'YOUR_DOCKERHUB_USERNAME/transport-dashboard', description: 'Docker image repository path')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Image Build Version Identifier')
    }

    stages {
        stage('Pull Code Structure') {
            steps {
                checkout scm
            }
        }

        stage('Parallel Quality Check') {
            parallel {
                stage('Code Quality Validation') {
                    steps {
                        echo "Validating HTML Syntax Layout..."
                        sh "grep -q 'city-transit' index.html || true"
                    }
                }
                stage('Dockerfile Validation') {
                    steps {
                        echo "Checking Container Directives..."
                        sh "grep -q 'FROM' Dockerfile"
                    }
                }
            }
        }

        stage('Build & Test Local Container') {
            steps {
                script {
                    sh "docker build -t ${params.DOCKER_IMAGE}:${params.IMAGE_TAG} ."
                    echo "Starting local execution tests..."
                    sh "docker run -d -p 8085:80 --name test_instance ${params.DOCKER_IMAGE}:${params.IMAGE_TAG}"
                    // Simple verification check to ensure application responds appropriately
                    sh "sleep 3 && curl -s http://localhost:8085 | grep -q 'Transit'"
                    // Cleanup local test containers
                    sh "docker stop test_instance && docker rm test_instance"
                }
            }
        }

        stage('Publish Image to Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker push ${params.DOCKER_IMAGE}:${params.IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploying Application Updates via Kubernetes') {
            steps {
                script {
                    // Update deployment template with active variables dynamically
                    sh "sed -i 's|DOCKERHUB_USERNAME/transport-dashboard:latest|${params.DOCKER_IMAGE}:${params.IMAGE_TAG}|g' deployment.yaml"
                    sh "kubectl apply -f deployment.yaml"
                    
                    echo "Checking Deployment Strategy Health..."
                    // Wait for rollouts to finish processing safely
                    sh "kubectl rollout status deployment/transport-dashboard --timeout=60s"
                }
            }
        }
    }
    
    post {
        failure {
            echo "Deployment Pipeline Issues Detected! Initiating Remediation Strategy..."
            sh "kubectl rollout undo deployment/transport-dashboard"
            echo "System Successfully Restored to the Previous Functional State."
        }
    }
}
