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
            docker.image('qnib/pytest').inside {
                sh '''
                echo "Running tests..."
                py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py
                '''
            }

            echo "Test stage completed. Checking test report..."
            junit 'test-reports/results.xml'  // Jalankan junit dengan path yang benar
        }

        stage('Deliver') {
    docker.image('cdrx/pyinstaller-linux:python2').inside {
        try {
            sh '''
            echo "Building executable..."
            pyinstaller --onefile sources/add2vals.py
            echo "Executable built successfully."
            '''
        } catch (Exception e) {
            echo "Error during build: ${e.getMessage()}"
            currentBuild.result = 'FAILURE'
            throw e
        }
    }
    echo "Archiving built artifact..."
    archiveArtifacts 'dist/add2vals'
}


    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        echo "Pipeline failed: ${e.getMessage()}"
    } finally {
        if (currentBuild.result == 'FAILURE') {
            echo "Pipeline failed! Here are the results:"
            sh 'cat ${WORKSPACE}/test-reports/results.xml'  // Output hasil tes jika gagal
        } else {
            echo "Pipeline succeeded!"
        }
    }
}
