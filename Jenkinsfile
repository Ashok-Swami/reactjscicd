pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
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

                    echo "Git Commit: ${GIT_COMMIT_SHORT}"
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t dreamy-frontend:${GIT_COMMIT_SHORT} \
                        .
                '''
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                    trivy image \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        dreamy-frontend:${GIT_COMMIT_SHORT}
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password \
                        --region "$AWS_REGION" | \
                    docker login \
                        --username AWS \
                        --password-stdin "$ECR_REGISTRY"
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                    docker tag \
                        dreamy-frontend:${GIT_COMMIT_SHORT} \
                        ${ECR_REGISTRY}/${ECR_REPOSITORY}:${GIT_COMMIT_SHORT}
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    docker push \
                        ${ECR_REGISTRY}/${ECR_REPOSITORY}:${GIT_COMMIT_SHORT}
                '''
            }
        }

        stage('Verify ECR') {
            steps {
                sh '''
                    aws ecr describe-images \
                        --repository-name "$ECR_REPOSITORY" \
                        --image-ids imageTag="$GIT_COMMIT_SHORT" \
                        --region "$AWS_REGION"
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD SUCCESS"
            echo "Git Tag: ${GIT_COMMIT_SHORT}"
        }

        failure {
            echo "CI/CD FAILED"
        }
    }
}