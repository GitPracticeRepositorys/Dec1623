pipeline {
    agent { label 'docker-node-1' }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/GitPracticeRepositorys/Dec1623.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("shivakrishna99/jenkinsdec23workshop:${BUILD_ID}")
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh "trivy image --format json -o trivy-report.json shivakrishna99/jenkinsdec23workshop:${BUILD_ID}"
                    publishHTML([reportName: 'Trivy Vulnerability Report', reportDir: '.', reportFiles: 'trivy-report.json', keepAll: true, alwaysLinkToLastBuild: true, allowMissing: false])
                }
            }
        }

        stage('Publish Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://your-docker-registry', 'docker-credentials-id') {
                        docker.image("shivakrishna99/jenkinsdec23workshop:${BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f deployment/k8s/deployment.yaml"
                sh "kubectl patch deployment netflix-app -p '{\"spec\":{\"template\":{\"spec\":{\"containers\":[{\"name\":\"netflix-app\",\"image\":\"shivakrishna99/jenkinsdec23workshop:${BUILD_ID}\"}]}}}}'"
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! You can add further steps here.'
        }
        failure {
            echo 'Pipeline failed! You can add further steps for failure handling.'
        }
    }
}
