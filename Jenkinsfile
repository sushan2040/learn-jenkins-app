pipeline {
    agent {
                docker {
                    image 'node:18-alpine'  // Docker image to use
                    reuseNode true  // Reuse the same workspace across stages (optional)
                }
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
