pipeline {
    agent any

    environment {
        IMAGE_NAME = "dreamy-frontend"
    }

    stages {

        stage('Checkout') {
            steps {
                echo '===== CHECKOUT CODE ====='

                git(
                    branch: 'main',
                    credentialsId: 'github-ssh',
                    url: 'git@github.com:Ashok-Swami/reactjscicd.git'
)
            }
        }

        stage('Git Information') {
            steps {
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short=8 HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Git Commit: ${env.GIT_COMMIT_SHORT}"
                }

                sh '''
                    echo "===== GIT INFORMATION ====="
                    git log -1 --oneline
                '''
            }
        }

        stage('Verify Frontend') {
            steps {
                sh '''
                    echo "===== PROJECT FILES ====="
                    ls -la

                    echo "===== PACKAGE.JSON ====="
                    cat package.json
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "===== DOCKER BUILD ====="

                    docker build \
                        -t ${IMAGE_NAME}:${GIT_COMMIT_SHORT} \
                        .
                '''
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                    echo "===== TRIVY SECURITY SCAN ====="

                    trivy image \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        ${IMAGE_NAME}:${GIT_COMMIT_SHORT}
                '''
            }
        }

        stage('Docker Verify') {
            steps {
                sh '''
                    echo "===== DOCKER IMAGE ====="

                    docker images ${IMAGE_NAME}

                    docker image inspect \
                        ${IMAGE_NAME}:${GIT_COMMIT_SHORT}
                '''
            }
        }
    }

    post {
        success {
            echo "======================================"
            echo "BUILD SUCCESSFUL"
            echo "Docker Image: ${IMAGE_NAME}:${GIT_COMMIT_SHORT}"
            echo "Security Scan: PASSED"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "BUILD FAILED"
            echo "======================================"
        }
    }
}