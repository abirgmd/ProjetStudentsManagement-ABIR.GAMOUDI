pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'abirgamoudi123/department-service'
        DOCKER_TAG   = 'latest'
    }

    options {
        timestamps()
    }

    stages {

        /* =======================
           📥 GIT CHECKOUT
           ======================= */
        stage('Git Checkout') {
            steps {
                echo "📥 Git Checkout"
                git branch: 'master',
                    url: 'https://github.com/abirgmd/ProjetStudentsManagement-ABIR.GAMOUDI.git'
            }
        }

        /* =======================
           🔨 MAVEN BUILD
           ======================= */
        stage('Clean & Build Maven') {
            steps {
                echo "🔨 Maven Build"
                sh '''
                    chmod +x mvnw
                    ./mvnw clean package -DskipTests
                '''
            }
        }

        /* =======================
           📊 SONARQUBE ANALYSIS
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

        /* =======================
           🐳 DOCKER BUILD
           ======================= */
        stage('Build Docker Image') {
            steps {
                echo "🐳 Docker Build"
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        /* =======================
           🔐 DOCKER PUSH
           ======================= */
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

                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }

        /* =======================
           ☸️ KUBERNETES DEPLOY
           ======================= */
        stage('Kubernetes Deploy') {
            steps {
                echo "☸️ Kubernetes Deploy (Minikube)"

                sh '''
                    kubectl cluster-info

                    kubectl get namespace devops || kubectl create namespace devops

                    kubectl apply -f mysql-deployment.yaml -n devops
                    kubectl apply -f spring-deployment.yaml -n devops

                    kubectl rollout status deployment spring-app -n devops --timeout=180s
                '''
            }
        }

        /* =======================
           🔍 VERIFY DEPLOYMENT
           ======================= */
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
            echo "✅ PIPELINE COMPLETED SUCCESSFULLY"
        }
        failure {
            echo "❌ PIPELINE FAILED"
        }
    }
}
