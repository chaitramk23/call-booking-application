pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    k8s-agent: dynamic
spec:
  serviceAccountName: jenkins-sa
  containers:
  - name: jnlp
    image: chaitramk23/jenkins-agent-eks:latest
    command:
    - cat
    tty: true
"""
        }
    }

    environment {
        UI_NAMESPACE = 'ui'
        API_NAMESPACE = 'api'
    }

    stages {
        stage('Checkout') {
            steps {
                container('jnlp') {
                    git branch: 'main', url: 'https://github.com/chaitramk23/call-booking-application.git'
                }
            }
        }

        stage('Deploy Admin UI') {
            steps {
                container('jnlp') {
                    sh "helm upgrade --install admin-ui ./charts/admin-ui -n $UI_NAMESPACE --create-namespace"
                }
            }
        }

        stage('Deploy User UI') {
            steps {
                container('jnlp') {
                    sh "helm upgrade --install user-ui ./charts/user-ui -n $UI_NAMESPACE --create-namespace"
                }
            }
        }

        stage('Deploy Admin API') {
            steps {
                container('jnlp') {
                    sh "helm upgrade --install admin-api ./charts/admin-api -n $API_NAMESPACE --create-namespace"
                }
            }
        }

        stage('Deploy User API') {
            steps {
                container('jnlp') {
                    sh "helm upgrade --install user-api ./charts/user-api -n $API_NAMESPACE --create-namespace"
                }
            }
        }
    }

    post {
        always {
            container('jnlp') {
                echo "Pipeline finished. Dynamic pod will be deleted automatically."
            }
        }
    }
}
