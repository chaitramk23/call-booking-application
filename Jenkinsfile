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
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    tty: true
  - name: k8s-agent
    image: lachlanevenson/k8s-helm:latest
    command:
      - cat
    tty: true
    volumeMounts:
      - name: workspace-volume
        mountPath: /home/jenkins/agent
  volumes:
    - name: workspace-volume
      emptyDir: {}
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
                container('k8s-agent') {
                    git branch: 'main', url: 'https://github.com/chaitramk23/call-booking-application.git'
                }
            }
        }

        stage('Deploy Admin UI') {
            steps {
                container('k8s-agent') {
                    sh "helm upgrade --install admin-ui ./charts/admin-ui -n \$UI_NAMESPACE --create-namespace"
                }
            }
        }

        stage('Deploy User UI') {
            steps {
                container('k8s-agent') {
                    sh "helm upgrade --install user-ui ./charts/user-ui -n \$UI_NAMESPACE --create-namespace"
                }
            }
        }

        stage('Deploy Admin API') {
            steps {
                container('k8s-agent') {
                    sh "helm upgrade --install admin-api ./charts/admin-api -n \$API_NAMESPACE --create-namespace"
                }
            }
        }

        stage('Deploy User API') {
            steps {
                container('k8s-agent') {
                    sh "helm upgrade --install user-api ./charts/user-api -n \$API_NAMESPACE --create-namespace"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Dynamic pod will be deleted automatically."
        }
    }
}
