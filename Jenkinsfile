pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME='learnjenkinsapp'
        AWS_DEFAULT_REGION="eu-north-1"
        AWS_DOCKER_REGISTRY='235494784067.dkr.ecr.eu-north-1.amazonaws.com'
        AWS_ECS_CLUSTER='LearnJenkinsApp-Cluster-Prod'
        AWS_ECS_SERVICE_PROD='LearnJenkinsApp-TaskDefination-Prod'
        AWS_ECS_TD_PROD='LearnJenkinsApp-TaskDefination-Prod'
    }

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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Build Docker image'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps{
               sh 'amazon-linux-extras install docker'
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
               
                sh '''
                docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                '''
                }
            }
        }
        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"

                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-defination-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_TD_PROD --task-definition $AWS_ECS_TD_PROD:2
                        aws ecs wait sservices-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD

                    '''
                }
            }
        }
        
    }
}