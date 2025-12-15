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
                git branch: 'master',
                    url: 'https://github.com/abirgmd/ProjetStudentsManagement-ABIR.GAMOUDI.git'
            }
        }

        /* =======================
           🔨 MAVEN BUILD
           ======================= */
        stage('Clean & Build Maven') {
            steps {
                sh '''
                    chmod +x mvnw
                    ./mvnw clean package -DskipTests
                '''
            }
        }

        /* =======================
           📊 SONARQUBE
           ======================= */
        stage('MVN SONARQUBE') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh './mvnw sonar:sonar'
                }
            }
        }

        /* =======================
           🐳 DOCKER BUILD
           ======================= */
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        /* =======================
           🔐 DOCKER PUSH
           ======================= */
        stage('Docker Login & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
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
                sh '''
                    kubectl get namespace devops || kubectl create namespace devops

                    kubectl apply -f mysql-deployment.yaml -n devops
                    kubectl apply -f spring-deployment.yaml -n devops

                    kubectl rollout status deployment spring-app -n devops --timeout=180s
                '''
            }
        }

        /* =======================
           📈 PROMETHEUS
           ======================= */
        stage('Prometheus') {
            steps {
                echo "📈 Starting Prometheus"
                sh 'docker start prometheus || true'
            }
        }

        /* =======================
           📊 GRAFANA
           ======================= */
        stage('Grafana') {
            steps {
                echo "📊 Starting Grafana"
                sh 'docker start grafana || true'
            }
        }

        /* =======================
           🔍 VERIFY DEPLOYMENT
           ======================= */
        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods -n devops
                    kubectl get svc -n devops
                '''
            }
        }
    }

    post {
        success {
            echo "✅ PIPELINE SUCCESSFULLY COMPLETED"
        }
        failure {
            echo "❌ PIPELINE FAILED"
        }
    }
}
