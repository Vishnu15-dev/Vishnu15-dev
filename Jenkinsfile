pipeline {
    agent any

    options {
        // Auto-cleanup old builds (keep only last 5)
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = 'katamreddyvishnu/sample-nginx'
        AWS_REGION = "us-east-1"
        CLUSTER_NAME = "Vishnu-demo"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/Vishnu15-dev/Vishnu15-dev.git', branch: 'master'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Update Deployment Manifest') {
            steps {
                script {
                    sh "sed -i 's|<DOCKER_IMAGE_TAG>|${IMAGE_NAME}:${BUILD_NUMBER}|g' nginx-deployment.yaml"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                        # Generate kubeconfig dynamically
                        aws eks update-kubeconfig \
                            --region $AWS_REGION \
                            --name $CLUSTER_NAME \
                            --kubeconfig kubeconfig.tmp

                        # Deploy manifests using kubeconfig
                        KUBECONFIG=kubeconfig.tmp kubectl apply -f nginx-deployment.yaml
                        KUBECONFIG=kubeconfig.tmp kubectl apply -f nginx-service.yaml
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Deployment failed. Check the logs."
        }
        success {
            echo "✅ Successfully deployed ${IMAGE_NAME}:${BUILD_NUMBER} to EKS!"
        }
    }
}
