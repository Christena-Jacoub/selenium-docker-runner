pipeline {
    agent any
    parameters { 
        choice choices: ['firefox', 'chrome', 'edge'], description: 'Choose browser', name: 'BROWSER'
        string defaultValue: 'vendor-portal', description: 'Add test suite name ', name: 'TEST_SUITE'  
        booleanParam defaultValue: false, name: 'BROWSERSTACK_ENABLED'
        booleanParam defaultValue: true, name: 'GRID_ENABLED'  
    }

    environment {
        BROWSERSTACK_USERNAME = credentials('BROWSERSTACK_CREDENTIALS') // Username
        BROWSERSTACK_ACCESS_KEY = credentials('BROWSERSTACK_CREDENTIALS') // Access key
    }

    stages {
        stage('Bring grid up') {
            steps {
                script {
                    if (params.GRID_ENABLED) {
                        // Start Docker grid (with browser service scaling)
                        sh "docker compose -f grid.yaml up --scale ${params.BROWSER}Service=2 -d"
                    } else {
                        echo "Running tests on BrowserStack..."
                        // Export credentials as environment variables for BrowserStack
                        sh """
                            export BROWSERSTACK_USERNAME=${BROWSERSTACK_USERNAME}
                            export BROWSERSTACK_ACCESS_KEY=${BROWSERSTACK_ACCESS_KEY}
                        """
                    }
                }
            }
        }

        stage('Run Test Suites') {
            steps {
                // Run tests with the given suite, passing necessary parameters
                sh "GRID_ENABLED=${params.GRID_ENABLED} BROWSERSTACK_ENABLED=${params.BROWSERSTACK_ENABLED} BROWSER=${params.BROWSER} TEST_SUITE=${params.TEST_SUITE} docker compose -f test-suites.yaml up --pull=always"
                
                script {
                    // Check if testng-failed.xml is found in output
                    if (fileExists('output/flight-reservation/testng-failed.xml') || fileExists('output/vendor-portal/testng-failed.xml')) {
                        error('Some tests failed')
                    }
                }
            }
        }
    }

    post {
        always {
            // Cleanup Docker containers after test execution
            sh "docker compose -f grid.yaml down"
            sh "docker compose -f test-suites.yaml down"

            // Archive the test results
            archiveArtifacts artifacts: 'output/flight-reservation/emailable-report.html', followSymlinks: false
            archiveArtifacts artifacts: 'output/vendor-portal/emailable-report.html', followSymlinks: false
        }
    }
}