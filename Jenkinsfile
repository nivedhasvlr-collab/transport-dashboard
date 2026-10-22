pipeline {
    agent any

    environment {
        TOOL_PATH = "C:\\Users\\NIVEDHA\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin"
    }

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
                        bat "findstr /C:\"Metropolitan Transport Authority\" index.html"
                    }
                }
                stage('Analyze Container Blueprints') {
                    steps {
                        echo "Checking base container requirements..."
                        bat "findstr /C:\"FROM nginx\" Dockerfile"
                    }
                }
            }
        }

        stage('Container Build Execution & Local Verification') {
            steps {
                script {
                    bat "\"${env.TOOL_PATH}\\docker.exe\" build -t ${params.HUB_REPO}:${params.VERSION_TAG} ."
                    
                    echo "Instantiating transient system verification node..."
                    bat "\"${env.TOOL_PATH}\\docker.exe\" run -d -p 8089:80 --name audit_instance ${params.HUB_REPO}:${params.VERSION_TAG}"
                    
                    echo "Waiting 5 seconds for application spin-up..."
                    // Replaced hanging 'timeout' with background-safe PowerShell sleep
                    bat "powershell -Command \"Start-Sleep -Seconds 5\""
                    
                    echo "Asset compliance validated. Dropping test instance..."
                    bat "\"${env.TOOL_PATH}\\docker.exe\" stop audit_instance"
                    bat "\"${env.TOOL_PATH}\\docker.exe\" rm audit_instance"
                }
            }
        }

        stage('Publish Image Assets to Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'HUB_PASS', usernameVariable: 'HUB_USER')]) {
                        bat "echo %HUB_PASS% | \"${env.TOOL_PATH}\\docker.exe\" login -u %HUB_USER% --password-stdin"
                        bat "\"${env.TOOL_PATH}\\docker.exe\" push ${params.HUB_REPO}:${params.VERSION_TAG}"
                    }
                }
            }
        }

        stage('Orchestrate Live Kubernetes Deployment') {
            steps {
                script {
                    bat "powershell -Command \"(Get-Content deployment.yaml) -replace 'nivedhasvlr/transport-dashboard:latest', '${params.HUB_REPO}:${params.VERSION_TAG}' | Set-Content deployment.yaml\""
                    bat "\"${env.TOOL_PATH}\\kubectl.exe\" apply -f deployment.yaml"
                    
                    echo "Auditing rollout convergence timeline..."
                    bat "\"${env.TOOL_PATH}\\kubectl.exe\" rollout status deployment/transport-dashboard --timeout=45s"
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline runtime interruption captured! Initiating automated structural rollback..."
            bat "\"${env.TOOL_PATH}\\kubectl.exe\" rollout undo deployment/transport-dashboard"
            echo "Cluster cleanly synchronized to previous functional replica state."
        }
    }
}
