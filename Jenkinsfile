pipeline {
    agent any

    stages {

        stage('Git Checkout') {
            steps {
                echo "📥 Git Checkout"
                git branch: 'master',
                    url: 'https://github.com/abirgmd/ProjetStudentsManagement-ABIR.GAMOUDI.git'
            }
        }

        stage('Build Maven') {
            steps {
                echo "🔨 Build Maven"
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                echo "🐳 Docker Build"
                sh '''
                    set -e
                    docker build -t abirgamoudi123/department-service:latest .
                '''
            }
        }

        stage('Load Image to Minikube') {
            steps {
                echo "📦 Load image to Minikube"
                sh 'minikube image load abirgamoudi123/department-service:latest'
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                echo "☸️ Kubernetes Deploy"
                sh 'kubectl apply -f mysql-deployment.yaml'
                sh 'kubectl apply -f spring-deployment.yaml'
                sh 'kubectl rollout status deployment spring-app'
            }
        }
    }

    post {
        success {
            echo "✅ PIPELINE OK"
        }
        failure {
            echo "❌ PIPELINE FAILED"
        }
    }
}
