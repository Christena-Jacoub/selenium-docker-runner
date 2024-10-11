pipeline {
    agent any
    stages {
        stage('Bring grid up') {
            steps {
                sh "docker compose -f grid.yaml up -d"
            }
        }
        stage('Run Tests suites') {
            steps {
               sh "docker compose -f test-suites.yaml up"
            }
        }
    }
    post{
        always{
           sh "docker compose -f grid.yaml down"
           sh "docker compose -f test-suites.yaml down"
           archiveArtifacts artifacts: 'output/flight-reservation/emailable-report.html', followSymlinks: false
           archiveArtifacts artifacts: 'output/vendor-portal/emailable-report.html', followSymlinks: false
        }
    }
}