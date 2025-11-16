pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello Aboura'
            }
        }
         stage('Récupération du projet depuis GitHub') {
            steps {
                echo 'Clonage du dépôt GitHub...'
                git branch: 'main', url: 'https://github.com/abirgmd/ProjetStudentsManagement-ABIR.GAMOUDI.git'
            }
        }

        stage('Compilation et génération du livrable') {
            steps {
                echo 'Compilation avec Maven...'
                sh 'mvn clean package'
            }
        }

    }
}

