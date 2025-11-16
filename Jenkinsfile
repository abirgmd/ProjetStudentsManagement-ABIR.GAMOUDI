pipeline {
    agent any

    tools {
        maven 'Maven' 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', 
                    url: 'https://github.com/abirgmd/ProjetStudentsManagement-ABIR.GAMOUDI.git'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package' 
            }
        }

        stage('Hello World') {
            steps {
                echo 'Hello World! Pipeline pour ProjetStudentsManagement'
            }
        }
    }

    post {
        success {
            echo 'Build terminé avec succès !'
        }
        failure {
            echo 'Build échoué ! Vérifie les logs.'
        }
    }
}
