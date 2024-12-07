node {
    // Tambahkan tahap untuk melakukan checkout dari repository Git
    stage('Checkout') {
        steps {
            // Mengambil kode dari repository yang sudah dikonfigurasi di Jenkins
            checkout scm
        }
    }

    // Tahap Build
    stage('Build') {
        docker.image('python:2-alpine').inside {
            // Periksa apakah file ada dan kemudian kompilasi
            sh '''
            echo "Checking files in sources directory..."
            ls -l sources/
            python -m py_compile sources/add2vals.py sources/calc.py
            '''
        }
    }
    
    // Tahap Test
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh '''
            echo "Running tests..."
            ls -l sources/
            py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py
            '''
        }
        post {
            always {
                junit 'test-reports/results.xml'
            }
        }
    }
    
    // Tahap Deliver
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
                archiveArtifacts 'dist/add2vals'
            }
        }
    }
}
