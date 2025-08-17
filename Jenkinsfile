pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        DOCKER_USER = 'yourdockerhubuser'
        IMAGE_NAME = 'nginx'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gvipinnair/new-jenkins.git'
                script {
                    def commitMsg = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
                    def matcher = commitMsg =~ /nginx:([\d\.]+)/
                    if (matcher) {
                        env.VERSION = matcher[0][1]
                        echo "Detected nginx version: ${env.VERSION}"
                    } else {
                        error("Commit message must contain version like nginx:1.23")
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${VERSION} .
                docker tag ${DOCKER_USER}/${IMAGE_NAME}:${VERSION} ${DOCKER_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                    echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                    docker push ${DOCKER_USER}/${IMAGE_NAME}:${VERSION}
                    docker push ${DOCKER_USER}/${IMAGE_NAME}:latest
                    docker logout
                    """
                }
            }
        }

        stage('Deploy Nginx to Kubernetes') {
            steps {
                sh """
                sed 's|image:.*|image: ${DOCKER_USER}/${IMAGE_NAME}:${VERSION}|' nginx-deployment.yaml > deploy-temp.yaml
                kubectl apply -f deploy-temp.yaml
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get pods -A'
                sh 'kubectl get svc'
            }
        }
    }
}