pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        DOCKER_IMAGE = 'yourdockerhubusername/nginx'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gvipinnair/new-jenkins.git'
            }
        }

        stage('Extract Version from Commit') {
            steps {
                script {
                    def commitMsg = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
                    echo "Commit message: ${commitMsg}"
                    def matcher = commitMsg =~ /nginx:([\d\.]+)/
                    if (matcher) {
                        env.VERSION = matcher[0][1]
                        echo "Using nginx version: ${env.VERSION}"
                    } else {
                        error("Commit message must contain nginx version like nginx:1.25.3")
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$VERSION .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE:$VERSION
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy Nginx to Kubernetes') {
            steps {
                sh '''
                sed "s|nginx:.*|$DOCKER_IMAGE:$VERSION|g" nginx-deployment.yaml > updated-nginx.yaml
                kubectl apply -f updated-nginx.yaml
                '''
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