pipeline {

    agent any

    environment {
        AWS_REGION     = "${env.AWS_REGION}"
        ECR_REGISTRY   = "${env.ECR_REGISTRY}"
        ECR_REPOSITORY = "${env.ECR_REPOSITORY}"

        EKS_CLUSTER_NAME = "dreamy-eks"
        IMAGE_NAME       = "dreamy-frontend"
        KUBECONFIG       = "/var/lib/jenkins/.kube/config"
    }

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

                    echo "Git Commit: ${env.GIT_COMMIT_SHORT}"
                }
            }
        }

        stage('Verify Frontend') {
            steps {
                sh '''
                    echo "Checking frontend project..."

                    test -f package.json
                    test -f Dockerfile

                    echo "Frontend files verified."
                '''
            }
        }

        stage('Verify Kubernetes Files') {
            steps {
                sh '''
                    echo "Checking Kubernetes manifests..."

                    test -f k8s/deployment.yaml
                    test -f k8s/service.yaml

                    echo "Kubernetes files verified."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${GIT_COMMIT_SHORT} \
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
                      ${IMAGE_NAME}:${GIT_COMMIT_SHORT}
                '''
            }
        }

        stage('Docker Verify') {
            steps {
                sh '''
                    docker images ${IMAGE_NAME}:${GIT_COMMIT_SHORT}
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password \
                      --region "$AWS_REGION" \
                    | docker login \
                      --username AWS \
                      --password-stdin "$ECR_REGISTRY"
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                    docker tag \
                      ${IMAGE_NAME}:${GIT_COMMIT_SHORT} \
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

        /*
         * =========================================
         * CD - AMAZON EKS
         * =========================================
         */

        stage('Verify EKS Tools') {
            steps {
                sh '''
                    echo "===== AWS CLI ====="
                    aws --version

                    echo "===== kubectl ====="
                    kubectl version --client

                    echo "===== EKS Cluster ====="

                    aws eks describe-cluster \
                      --name "$EKS_CLUSTER_NAME" \
                      --region "$AWS_REGION" \
                      --query 'cluster.status' \
                      --output text
                '''
            }
        }

        stage('Configure kubectl') {
            steps {
                sh '''
                    echo "Updating EKS kubeconfig..."

                    aws eks update-kubeconfig \
                      --name "$EKS_CLUSTER_NAME" \
                      --region "$AWS_REGION" \
                      --kubeconfig "$KUBECONFIG"

                    echo "===== EKS Nodes ====="

                    kubectl \
                      --kubeconfig "$KUBECONFIG" \
                      get nodes
                '''
            }
        }

        stage('Prepare Kubernetes Manifest') {
            steps {
                sh '''
                    echo "Updating image in Kubernetes deployment..."

                    sed -i \
                      "s|IMAGE_PLACEHOLDER|${ECR_REGISTRY}/${ECR_REPOSITORY}:${GIT_COMMIT_SHORT}|g" \
                      k8s/deployment.yaml

                    echo "===== Deployment Image ====="

                    grep "image:" k8s/deployment.yaml
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    echo "======================================"
                    echo "DEPLOYING KUBERNETES RESOURCES"
                    echo "======================================"

                    echo "Applying ConfigMap..."
                    kubectl \
                    --kubeconfig "$KUBECONFIG" \
                    apply -f k8s/configmap.yaml

                    echo "Applying Deployment..."
                    kubectl \
                    --kubeconfig "$KUBECONFIG" \
                    apply -f k8s/deployment.yaml

                    echo "Applying Service..."
                    kubectl \
                    --kubeconfig "$KUBECONFIG" \
                    apply -f k8s/service.yaml

                    echo "Applying HPA..."
                    kubectl \
                    --kubeconfig "$KUBECONFIG" \
                    apply -f k8s/hpa.yaml

                    echo "======================================"
                    echo "KUBERNETES RESOURCES APPLIED"
                    echo "======================================"
                '''
            }
        }
        stage('Wait for Deployment') {
            steps {
                sh '''
                    echo "Waiting for deployment rollout..."

                    kubectl \
                      --kubeconfig "$KUBECONFIG" \
                      rollout status \
                      deployment/dreamy-frontend \
                      --timeout=5m
                '''
            }
        }

stage('Verify Deployment') {
    steps {
        sh '''
            echo "======================================"
            echo "PODS"
            echo "======================================"

            kubectl \
              --kubeconfig "$KUBECONFIG" \
              get pods -o wide

            echo "======================================"
            echo "DEPLOYMENT"
            echo "======================================"

            kubectl \
              --kubeconfig "$KUBECONFIG" \
              get deployment dreamy-frontend

            echo "======================================"
            echo "SERVICE"
            echo "======================================"

            kubectl \
              --kubeconfig "$KUBECONFIG" \
              get service dreamy-frontend

            echo "======================================"
            echo "HPA"
            echo "======================================"

            kubectl \
              --kubeconfig "$KUBECONFIG" \
              get hpa dreamy-frontend

            echo "======================================"
            echo "CONFIGMAP"
            echo "======================================"

            kubectl \
              --kubeconfig "$KUBECONFIG" \
              get configmap dreamy-frontend-config

            echo "======================================"
        '''
    }
}
    }

    post {

        success {
            echo "======================================"
            echo "CI/CD PIPELINE SUCCESS"
            echo "======================================"

            echo "Git Commit:"
            echo "${GIT_COMMIT_SHORT}"

            echo "ECR Image:"
            echo "${ECR_REGISTRY}/${ECR_REPOSITORY}:${GIT_COMMIT_SHORT}"

            echo "EKS Cluster:"
            echo "${EKS_CLUSTER_NAME}"

            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "CI/CD PIPELINE FAILED"
            echo "======================================"

            echo "Check the failed stage in the Jenkins console."

            echo "======================================"
        }
    }
}