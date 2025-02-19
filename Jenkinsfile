pipeline {
    agent {
        docker { image 'node:22.14.0-alpine3.21' }
    }
    stages {
        stage('Build') {
           
            steps {
                script {
                    // Debugging steps: check files and node/npm versions
                    sh '''
                        echo "Listing files"
                        ls -la
                        echo "Node Version:"
                        node --version
                        echo "NPM Version:"
                        npm --version
                        
                        echo "Running npm ci"
                        npm ci

                        echo "Building the project"
                        npm run build

                        echo "Listing files after build"
                        ls -la
                    '''
                }
            }
        }
    }
}
