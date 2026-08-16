pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_PUBLIC_REPO_URI = 'public.ecr.aws/j6a4o3t5/jenkinsecr'
        IMAGE_TAG = 'latest'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
        EKS_CLUSTER = 'my-eks-cluster'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'

                git branch: 'main',
                    url: 'https://github.com/Ajayrichard/Jenkinsandjava.git'
            }
        }

        stage('Verify AWS and EKS') {
            steps {
                sh '''
                    echo "AWS Identity:"
                    aws sts get-caller-identity

                    echo "EKS Nodes:"
                    kubectl get nodes
                '''
            }
        }

        stage('Build Java Application') {
            steps {
                echo 'Building Java application...'

                sh '''
                    mvn clean -B -Denforcer.skip=true package
                '''
            }
        }

        stage('Login to ECR Public') {
            steps {
                echo 'Logging into ECR Public...'

                sh '''
                    aws ecr-public get-login-password \
                        --region us-east-1 |
                    docker login \
                        --username AWS \
                        --password-stdin public.ecr.aws
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build -t ${IMAGE_URI} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to ECR...'

                sh '''
                    docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo 'Deploying application to EKS...'

                sh '''
                    aws eks update-kubeconfig \
                        --region ${AWS_REGION} \
                        --name ${EKS_CLUSTER}

                    kubectl apply -f deploymentjava.yaml

                    kubectl apply -f servicelb.yaml

                    echo "Pods:"
                    kubectl get pods

                    echo "Services:"
                    kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'PIPELINE SUCCESSFUL'
            echo 'Docker image pushed to ECR'
            echo 'Application deployed to EKS'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'PIPELINE FAILED'
            echo 'Check Console Output'
            echo '======================================'
        }
    }
}
