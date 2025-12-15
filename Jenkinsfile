pipeline {
    agent any
  
    stages {

        stage('Git') {
            steps {
                script {
                    git credentialsId: 'github-credentials',
                        branch: 'master',
                        url: 'https://github.com/abirgmd/ProjetStudentsManagement-ABIR.GAMOUDI.git'
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'chmod +x mvnw && ./mvnw clean install'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh './mvnw test'
            }
        }
        
        stage('Build DockerImage') {
            steps {
                sh 'docker build -t abirgamoudi123/department-service:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKER_USERNAME',
                            passwordVariable: 'DOCKER_PASSWORD'
                        )
                    ]) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            docker push abirgamoudi123/department-service:latest
                        '''
                    }
                }
            }
        }

        stage('Docker Compose') {
            steps {
                sh 'docker-compose up -d'
            }
        }

        stage('Jacoco Static Analysis') {
            steps {
                junit 'target/surefire-reports/**/*.xml'
                jacoco()
            }
        }

        stage('MVN SONARQUBE') {
            steps {
                withCredentials([
                    string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')
                ]) {
                    sh '''
                        ./mvnw sonar:sonar \
                        -Dsonar.login=$SONAR_TOKEN \
                        -Dsonar.host.url=http://192.168.33.10:9000
                    '''
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                sh './mvnw deploy'
            }
        }

        stage('Prometheus') {
            steps {
                sh 'docker start prometheus || true'
            }
        }

        stage('Grafana') {
            steps {
                sh 'docker start grafana || true'
            }
        }

        stage('Terraform') {
            steps {
                sh 'terraform init'
                sh 'terraform apply -auto-approve'
            }
        }
    }
       
    post {
        success {
            emailext(
                subject: "Build Success: ${currentBuild.fullDisplayName}",
                body: "Le pipeline a réussi. Voir les détails du build ici: ${env.BUILD_URL}",
                to: 'abir.gamoudi@esprit.tn'
            )
        }
        failure {
            emailext(
                subject: "Build Failed: ${currentBuild.fullDisplayName}",
                body: "Le pipeline a échoué. Voir les détails du build ici: ${env.BUILD_URL}",
                to: 'abir.gamoudi@esprit.tn'
            )
        }
    }
}
