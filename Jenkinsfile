pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "=== Building the project ==="
                    ls -la 
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "=== Running tests ==="
                    node --version
                    npm test || echo "⚠️ Tests failed, but pipeline continues for debugging"
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished — archiving workspace contents"
            archiveArtifacts artifacts: '**/*', allowEmptyArchive: true
        }
    }
}
