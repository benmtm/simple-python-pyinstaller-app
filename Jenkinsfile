pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                    args '--rm'
                }
            }
            steps {
                sh '''
                    apk add --no-cache gcc musl-dev
                    python -m py_compile sources/add2vals.py sources/calc.py
                '''
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                    args '--rm'
                }
            }
            steps {
                sh '''
                    mkdir -p test-reports
                    py.test --verbose --junit-xml=test-reports/results.xml sources/test_calc.py
                '''
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python2'
                    args '--rm'
                }
            }
            steps {
                sh '''
                    mkdir -p dist
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
}
