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
        docker.image('qnib/pytest').inside('-u root') {
            sh '''
            echo "Running tests..."
            mkdir -p test-reports
            py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py
            ls -l test-reports/
            '''
        }
        post {
            always {
                echo "Test stage completed. Checking test report..."
                sh 'ls -l test-reports/'
                junit 'test-reports/results.xml'
            }
        }
    }
    
    stage('Deliver') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh '''
            echo "Building executable..."
            ls -l sources/
            pyinstaller --onefile sources/add2vals.py
            '''
        }
        post {
            success {
                echo "Archiving artifact..."
                archiveArtifacts 'dist/add2vals'
            }
        }
    }
}
