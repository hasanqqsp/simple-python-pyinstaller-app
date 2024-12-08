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
            junit 'test-reports/results.xml'  
        }
        stage('Manual Approval') {
            input "Lanjutkan ke tahap Deploy?"
        }
       stage('Deploy') {
            docker.image('python:2-slim').inside('-u root') {
                sh '''
                echo "Installing PyInstaller..."
                apt-get update && apt-get install -y build-essential libffi-dev
                pip install pyinstaller==3.6
                echo "Building executable..."
                pyinstaller --onefile sources/add2vals.py
                '''
            }

            echo "Archiving built artifact..."
            sleep(60)
            archiveArtifacts 'dist/add2vals'
            echo "Copying artifact to remote server..."
            sshPublisher(publishers: [
                sshPublisherDesc(configName: 'deployment-server', transfers: [
                    sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: true, remoteDirectory: 'dicoding-ci-cd', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'dist/add2vals'),
                    sshTransfer(cleanRemote: false, excludes: '', execCommand: 'cd /home/ubuntu/dicoding-ci-cd/dist && chmod +x add2vals && ./add2vals 2 5 && ./add2vals 5', execTimeout: 120000, flatten: false, makeEmptyDirs: false, remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')
                ], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)
            ])
        
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        echo "Pipeline failed: ${e.getMessage()}"
    } finally {
        if (currentBuild.result == 'FAILURE') {
            echo "Pipeline failed! Here are the results:"
            sh 'cat ${WORKSPACE}/test-reports/results.xml' 
        } else {
            echo "Pipeline succeeded!"
        }
    }
}
