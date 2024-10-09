pipeline {
    agent any
    stages {
        stage('Bring grid up and run the test') {
            steps {
                sh 'docker compose up'
            }
        }
        stage('Bring grid down') {
            steps {
               sh 'docker compose down'
            }
        }
    }
}