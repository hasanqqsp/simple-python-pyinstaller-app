node {
    try {
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

            echo "Test stage completed. Checking test report..."
            sh 'ls -l ${WORKSPACE}/test-reports/'
            junit "${WORKSPACE}/test-reports/results.xml"
        }

        stage('Deliver') {
            docker.image('cdrx/pyinstaller-linux:python2').inside {
                sh '''
                echo "Building executable..."
                pyinstaller --onefile sources/add2vals.py
                '''
            }

            echo "Archiving built artifact..."
            archiveArtifacts 'dist/add2vals'
        }
        
    } catch (Exception e) {
        // Handle failure
        currentBuild.result = 'FAILURE'
        echo "Pipeline failed: ${e.getMessage()}"
    } finally {
        // Always execute this block, regardless of success or failure
        if (currentBuild.result == 'FAILURE') {
            echo "Pipeline failed! Here are the results:"
            sh 'cat ${WORKSPACE}/test-reports/results.xml'
        } else {
            echo "Pipeline succeeded!"
        }
    }
}
