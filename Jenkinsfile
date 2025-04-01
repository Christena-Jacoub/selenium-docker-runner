pipeline {
    agent any
    parameters { 
        choice choices: ['firefox', 'chrome', 'edge'], description: 'Choose browser', name: 'BROWSER'
        string defaultValue: 'vendor-portal', description: 'Add test suite name ', name: 'TEST_SUITE'  
        booleanParam defaultValue: false, name: 'BROWSERSTACK_ENABLED'
        booleanParam defaultValue: true, name: 'GRID_ENABLED'  
    }

    environment {
        BROWSERSTACK_USERNAME = credentials('BROWSERSTACK_CREDENTIALS_USR')
        BROWSERSTACK_ACCESS_KEY = credentials('BROWSERSTACK_CREDENTIALS_PSW')
    }
    stages {
        stage('Bring grid up') {
            steps {
                script{
                    if(params.GRID_ENABLED){
                        sh "docker compose -f grid.yaml up --scale ${params.BROWSER}Service=2 -d" // this is the same as this is to make 2 containers same like: docker compose -f grid.yaml up --scale chromeService=2 -d
                    }
                    else{
                        echo "Running tests on BrowserStack..."
                        sh '''
                        export BROWSERSTACK_USERNAME=${BROWSERSTACK_USERNAME}
                        export BROWSERSTACK_ACCESS_KEY=${BROWSERSTACK_ACCESS_KEY}
                        '''
                    }
                }
               
            }
        }
        stage('Run Tests suites') {
            steps {
                // same as: BROWSER=chrome docker compose up
                // add option --pull=always to get the latest image every time before running it especially when many dev are working on the same solution
                // pass TEST_SUITE as a parameter
               sh "GRID_ENABLED=${params.GRID_ENABLED} BROWSERSTACK_ENABLED=${params.BROWSERSTACK_ENABLED} BROWSER=${params.BROWSER} TEST_SUITE=${param.TEST_SUITE} docker compose -f test-suites.yaml up --pull=always"
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
