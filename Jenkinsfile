pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'abirgamoudi123/department-service'
        DOCKER_TAG   = 'latest'
    }

    tools {
        maven 'Maven'
    }

    options {
        timestamps()
    }

    stages {

        stage('Git Checkout') {
            steps {
                echo "📥 Git Checkout"
                git branch: 'master',
                    url: 'https://github.com/abirgmd/ProjetStudentsManagement-ABIR.GAMOUDI.git'
            }
        }

        stage('Clean & Build Maven') {
            steps {
                echo "🔨 Build Maven"
                sh '''
                    chmod +x mvnw
                    ./mvnw clean package -DskipTests
                '''
            }
        }

        /* =======================
           🔍 SONARQUBE ANALYSIS
           ======================= */
        stage('MVN SONARQUBE') {
            steps {
                echo "📊 SonarQube Analysis"
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        ./mvnw sonar:sonar
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Docker Build"
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        stage('Docker Login & Push') {
            steps {
                echo "🔐 Docker Login & Push"

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        export DOCKER_CLIENT_TIMEOUT=300
                        export COMPOSE_HTTP_TIMEOUT=300

                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        for i in 1 2 3; do
                          echo "Docker push attempt $i..."
                          docker push ${DOCKER_IMAGE}:${DOCKER_TAG} && break
                          echo "Retrying in 10 seconds..."
                          sleep 10
                        done
                    '''
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                echo "☸️ Kubernetes Deploy (Minikube Local)"

                sh '''
                    echo "📌 Cluster info"
                    kubectl cluster-info

                    echo "📌 Ensure namespace devops"
                    kubectl get namespace devops || kubectl create namespace devops

                    echo "📌 Deploy MySQL"
                    kubectl apply -f mysql-deployment.yaml -n devops

                    echo "📌 Deploy Spring App"
                    kubectl apply -f spring-deployment.yaml -n devops

                    echo "📌 Wait for rollout"
                    kubectl rollout status deployment spring-app -n devops --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "🔍 Verify Deployment"
                sh '''
                    kubectl get pods -n devops
                    kubectl get svc -n devops
                    kubectl get deployments -n devops
                '''
            }
        }
    }

    post {
        success {
            echo "✅ PIPELINE ABIR COMPLETED SUCCESSFULLY"
        }
        failure {
            echo "❌ PIPELINE ABIR FAILED"
        }
    }
}
