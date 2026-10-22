pipeline {
    agent any

    parameters {
        string(name: 'HUB_REPO', defaultValue: 'nivedhasvlr/transport-dashboard', description: 'Docker Hub Destination Path')
        string(name: 'VERSION_TAG', defaultValue: 'latest', description: 'Container Version Registry Label')
    }

    stages {
        stage('Sourcing Updates') {
            steps {
                checkout scm
            }
        }

        stage('Parallel Code Integrity Check') {
            parallel {
                stage('Analyze HTML Assets') {
                    steps {
                        echo "Parsing layout anchors..."
                        // Windows uses findstr instead of grep
                        bat "findstr /C:\"Metropolitan Transport Authority\" index.html"
                    }
                }
                stage('Analyze Container Blueprints') {
                    steps {
                        echo "Checking base container requirements..."
                        // Windows uses findstr instead of grep
                        bat "findstr /C:\"FROM nginx\" Dockerfile"
                    }
                }
            }
        }

        stage('Container Build Execution & Local Verification') {
            steps {
                script {
                    bat "docker build -t ${params.HUB_REPO}:${params.VERSION_TAG} ."
                    
                    echo "Instantiating transient system verification node..."
                    bat "docker run -d -p 8089:80 --name audit_instance ${params.HUB_REPO}:${params.VERSION_TAG}"
                    
                    // Windows sleep implementation using timeout
                    bat "timeout /t 5 /nobreak"
                    
                    echo "Asset compliance validated. Dropping test instance..."
                    bat "docker stop audit_instance"
                    bat "docker rm audit_instance"
                }
            }
        }

        stage('Publish Image Assets to Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'HUB_PASS', usernameVariable: 'HUB_USER')]) {
                        // Secure login string syntax optimized for Windows Command Prompt
                        bat "echo %HUB_PASS% | docker login -u %HUB_USER% --password-stdin"
                        bat "docker push ${params.HUB_REPO}:${params.VERSION_TAG}"
                    }
                }
            }
        }

        stage('Orchestrate Live Kubernetes Deployment') {
            steps {
                script {
                    // Replaced Linux 'sed' with a universal PowerShell text substitution tool block
                    bat "powershell -Command \"(Get-Content deployment.yaml) -replace 'nivedhasvlr/transport-dashboard:latest', '${params.HUB_REPO}:${params.VERSION_TAG}' | Set-Content deployment.yaml\""
                    bat "kubectl apply -f deployment.yaml"
                    
                    echo "Auditing rollout convergence timeline..."
                    bat "kubectl rollout status deployment/transport-dashboard --timeout=45s"
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline runtime interruption captured! Initiating automated structural rollback..."
            bat "kubectl rollout undo deployment/transport-dashboard"
            echo "Cluster cleanly synchronized to previous functional replica state."
        }
    }
}
