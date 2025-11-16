pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello Aboura'
            }
        }

        stage('Compilation et génération du livrable') {
            steps {
                echo 'Compilation avec Maven...'
                sh 'mvn clean package'
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès ! Le livrable se trouve dans target/.'
        }
        failure {
            echo 'Le pipeline a échoué. Vérifie les erreurs ci-dessus.'
        }
    }
}
