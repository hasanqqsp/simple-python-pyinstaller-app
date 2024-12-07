node {
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }
    
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh '''
            echo "Running tests..."
            mkdir -p test-reports
            py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py
            '''
        }
        post {
            always {
                echo "Test stage completed. Checking test report..."
                sh 'ls -l ${WORKSPACE}/test-reports/'
                junit '${WORKSPACE}/test-reports/results.xml'
            }
        }
    }
    
    stage('Deliver') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh 'pyinstaller --onefile sources/add2vals.py'
        }
        post {
            success {
                archiveArtifacts 'dist/add2vals'
            }
        }
    }
}
