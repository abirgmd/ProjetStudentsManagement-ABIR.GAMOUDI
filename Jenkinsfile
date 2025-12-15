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
           🔨 MAVEN BUILD + TESTS
           ======================= */
        stage('Build & Test (JUnit)') {
            steps {
                echo "🧪 Maven Build & Tests"
                sh '''
                    chmod +x mvnw
                    ./mvnw clean test
                '''
            }
        }

        /* =======================
           📊 JACOCO COVERAGE
           ======================= */
        stage('JaCoCo Coverage') {
            steps {
                echo "📊 JaCoCo Report"
                sh './mvnw jacoco:report'
                jacoco execPattern: 'target/jacoco.exec'
            }
        }

        /* =======================
           📊 SONARQUBE
           ======================= */
        stage('SonarQube Analysis') {
            steps {
                echo "📊 SonarQube Analysis"
                withSonarQubeEnv('SonarQube') {
                    sh './mvnw sonar:sonar'
                }
            }
        }

        /* =======================
           🐳 DOCKER BUILD
           ======================= */
        stage('Docker Build') {
            steps {
                echo "🐳 Docker Build"
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        /* =======================
           🔐 DOCKER PUSH
           ======================= */
        stage('Docker Push') {
            steps {
                echo "🔐 Docker Push"
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
                echo "☸️ Kubernetes Deploy"
                sh '''
                    kubectl get namespace devops || kubectl create namespace devops
                    kubectl apply -f mysql-deployment.yaml -n devops
                    kubectl apply -f spring-deployment.yaml -n devops
                    kubectl rollout status deployment spring-app -n devops --timeout=180s
                '''
            }
        }

        /* =======================
           🏗️ TERRAFORM (INFRA)
           ======================= */
        stage('Terraform Apply') {
            steps {
                echo "🏗️ Infrastructure Provisioning"
                dir('terraform') {
                    sh '''
                        terraform init
                        terraform apply -auto-approve
                    '''
                }
            }
        }

        /* =======================
           📈 PROMETHEUS
           ======================= */
        stage('Prometheus') {
            steps {
                echo "📈 Start Prometheus"
                sh 'docker start prometheus || true'
            }
        }

        /* =======================
           📊 GRAFANA
           ======================= */
        stage('Grafana') {
            steps {
                echo "📊 Start Grafana"
                sh 'docker start grafana || true'
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
                '''
            }
        }
    }

    post {
        success {
            echo "✅ PIPELINE ABIR CI/CD + INFRA + OBSERVABILITY SUCCESS"
        }
        failure {
            echo "❌ PIPELINE ABIR FAILED"
        }
    }
}
