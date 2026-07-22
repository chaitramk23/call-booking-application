# call-booking-application



![ma](https://user-images.githubusercontent.com/29688323/179655923-e5d9ed72-176e-4956-897c-c1bb434d5c63.jpg)

## Project Directory Structure

```text
.
call-booking-application/
├── README.md
├── PROJECT_STRUCTURE.md
├── Jenkinsfile
├── docker-compose.yaml
├── admin-api/
├── admin-ui/
├── user-api/
├── user-ui/
├── charts/
├── kubernetes/
└── Docs/  
```



pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/chaitramk23/call-booking-application.git'
        KUBE_CONFIG = '/home/jenkins/.kube/config' // path to kubeconfig
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Set Namespaces') {
            steps {
                sh """
                kubectl create namespace ui --dry-run=client -o yaml | kubectl apply -f -
                kubectl create namespace api --dry-run=client -o yaml | kubectl apply -f -
                """
            }
        }

        stage('Deploy UI Microservices') {
            steps {
                sh """
                helm upgrade --install admin-ui charts/admin-ui -n ui
                helm upgrade --install user-ui charts/user-ui -n ui
                """
            }
        }

        stage('Deploy API Microservices') {
            steps {
                sh """
                helm upgrade --install admin-api charts/admin-api -n api
                helm upgrade --install user-api charts/user-api -n api
                """
            }
        }

    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
