node {
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh '''
            echo "Building source files..."
            python -m py_compile sources/add2vals.py sources/calc.py
            '''
        }
    }
    
    stage('Test') {
        // Menjalankan container dengan volume berbagi ke direktori WORKSPACE
        docker.image('qnib/pytest').inside("-v ${WORKSPACE}:/workspace") {
            sh '''
            echo "Running tests..."
            mkdir -p /workspace/test-reports
            py.test --verbose --junit-xml /workspace/test-reports/results.xml sources/test_calc.py
            '''
        }
        post {
            always {
                echo "Test stage completed. Checking test report..."
                sh 'ls -l ${WORKSPACE}/test-reports/'
                junit '${WORKSPACE}/test-reports/results.xml'
            }
            failure {
                echo "Tests failed! Here are the results:"
                sh 'cat ${WORKSPACE}/test-reports/results.xml'
            }
        }
    }
    
    stage('Deliver') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh '''
            echo "Building executable..."
            pyinstaller --onefile sources/add2vals.py
            '''
        }
        post {
            success {
                echo "Archiving built artifact..."
                archiveArtifacts 'dist/add2vals'
            }
            failure {
                echo "Build failed! Check logs for errors."
            }
        }
    }
}
