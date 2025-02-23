pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID='58049706-f6cc-4ea0-b0aa-2c1d911b30f8'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
    }

    stages {
        
        stage('Docker'){
            steps{
                sh 'docker build -t my-playwright .'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Deploy staging'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
                steps{
                    sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify netlify --version
                    echo deploying to producttion site  id:$NETLIFY_SITE_ID
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                    echo 'added to check git polling'
                    '''
                script{
                    env.STAGING_URL=sh(script:"node_modules/.bin/node-jq -r'.deploy_url' deploy-output.json",returnStdout:true);
                }
                }
                
            }
             stage('staging E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment{
                         CI_ENVIRONMENT_URL="${env.STAGING_URL}"
                         }

                    steps {
                        sh '''
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                } 
         stage('Approval'){
            steps{
                timeout(activity: true, time: 15,unit:'MINUTES') {
                // some block
                }
                input message: 'Do you want to deploy to production?', ok: 'Yes I want to deploy'
            }
         }   
        stage('Deploy prod'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
                steps{
                    sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify netlify --version
                    echo deploying to producttion site  id:$NETLIFY_SITE_ID
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    echo 'added to check git polling'
                    '''
                }
            }
             stage('prod E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment{
                         CI_ENVIRONMENT_URL='https://lucky-bunny-deb54c.netlify.app'
                         }

                    steps {
                        sh '''
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }       

        // stage('Tests') {
        //     parallel {
        //         stage('Unit tests') {
        //             agent {
        //                 docker {
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }

        //             steps {
        //                 sh '''
        //                     #test -f build/index.html
        //                     npm test
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }

        //         stage('E2E') {
        //             agent {
        //                 docker {
        //                     image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //                     reuseNode true
        //                 }
        //             }

        //             steps {
        //                 sh '''
        //                     npm install serve
        //                     node_modules/.bin/serve -s build &
        //                     sleep 10
        //                     npx playwright test  --reporter=html
        //                 '''
        //             }

        //             post {
        //                 always {
        //                     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        //                 }
        //             }
        //         }
        //     }
        // }
    }
}