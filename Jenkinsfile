pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/gvipinnair/new-jenkins.git' 
            }
        }

        stage('Deploy Nginx to Kubernetes') {
            steps {
                sh 'kubectl apply -f nginx-deployment.yaml'
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