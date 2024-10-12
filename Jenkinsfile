pipeline {
    agent any
    parameters { // this is the same as environment variable however we can't change their values inside the stages
        choice choices: ['firefox', 'chrome', 'edge'], description: 'Choose browser', name: 'BROWSER'
    }
    stages {
        stage('Bring grid up') {
            steps {
                // this is to make 2 containers same like: docker compose -f grid.yaml up --scale chromeService=2 -d
                sh "docker compose -f grid.yaml up --scale ${params.BROWSER}Service=2 -d" // this is the same as
            }
        }
        stage('Run Tests suites') {
            steps {
                // same as: BROWSER=chrome docker compose up
                // add option --pull=always to get the latest image every time before running it especially when many dev are working on the same solution
               sh "BROWSER=${params.BROWSER} docker compose -f test-suites.yaml up --pull=always"
               // this script in groovy and let Jenkins checks if any testng-failed.xml is found in the output dir, it will mark the tests as failed Jenkins
               script{
                    if(fileExists('output/flight-reservation/testng-failed.xml')||fileExists('output/vendor-portal/testng-failed.xml')){
                        error ('Some Tests are failed')
                    }    
               }
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