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

        stage('Git Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/abirgmd/ProjetStudentsManagement-ABIR.GAMOUDI.git'
            }
        }

        stage('Clean & Build Maven') {
            steps {
                sh '''
                    chmod +x mvnw
                    ./mvnw clean package -DskipTests
                '''
            }
        }

        stage('MVN SONARQUBE') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh './mvnw sonar:sonar'
                }
            }
        }

        // les autres stages Docker / Kubernetes restent inchangés
    }
}
