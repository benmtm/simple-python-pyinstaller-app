pipeline {
    agent none  // Tidak menggunakan agent global, akan dideklarasikan di tiap stage
    stages {
        stage('Setup Docker') {
            steps {
                sh '''
                    echo "Checking if Docker is installed..."
                    if ! command -v docker &> /dev/null
                    then
                        echo "Docker not found, installing Docker..."
                        apt-get update && apt-get install -y docker.io
                    else
                        echo "Docker is already installed"
                    fi
                '''
            }
        }

        stage('Setup Environment') {
            agent {
                docker {
                    image 'alpine'
                    args '--rm' // Hapus container setelah selesai
                }
            }
            steps {
                sh '''
                    echo "Setting up environment..."
                    apk add --no-cache python3 py3-pip
                    python3 --version
                '''
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'python:3-alpine' // Gunakan Python 3 pada Alpine
                    args '--rm' // Hapus container setelah selesai
                }
            }
            steps {
                sh '''
                    echo "Starting build..."
                    mkdir -p build
                    python -m py_compile sources/add2vals.py sources/calc.py
                    echo "Build completed!"
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'pytest/pytest' // Gunakan image pytest untuk testing
                    args '--rm' // Hapus container setelah selesai
                }
            }
            steps {
                sh '''
                    echo "Running tests..."
                    mkdir -p test-reports
                    py.test --verbose --junit-xml=test-reports/results.xml sources/test_calc.py
                '''
            }
            post {
                always {
                    junit 'test-reports/results.xml' // Publikasikan hasil tes
                }
            }
        }

        stage('Deliver') {
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python3' // Image untuk PyInstaller
                    args '--rm' // Hapus container setelah selesai
                }
            }
            steps {
                sh '''
                    echo "Packaging application..."
                    pyinstaller --onefile sources/add2vals.py
                    echo "Packaging completed!"
                '''
            }
            post {
                success {
                    echo "Archiving build artifacts..."
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed!'
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
