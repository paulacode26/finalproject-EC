pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'eu-north-1'
        AWS_DOCKER_REGISTRY = '608232564950.dkr.ecr.us-east-2.amazonaws.com'
        // your ECR repository name
        APP_NAME = 'group-project-ce'
        
    }
    stages {
        stage('Build') {
            agent{
                docker {
                    image 'node:20.15.1-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la 
                    node --version
                    npm --version
                    npm install
                    npm run build
                    ls -la 
                '''
            }
        }
        stage('Test') {
            agent{
                docker{
                    image 'node:20.15.1-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        } 

        stage('Build My Docker Image'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'CE_Group1', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    
                    sh '''
                        amazon-linux-extras install docker
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:latest
                    '''
                }
            }
        } 

        // stage('Deploy to AWS') {
        //      agent {
        //          docker {
        //              image 'amazon/aws-cli'
        //              reuseNode true
        //              args '-u root --entrypoint=""'
        //          }
        //     }
           
        //      steps {
        //         withCredentials([usernamePassword(credentialsId: 'finalprojectNewUserKey', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')])
        //         {  
        //              sh '''
        //                  aws --version
        //                 yum install jq -y
                       
        //                  LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
        //                  aws ecs update-service --cluster my-group-project-ce --service my-group-project-ce-service --task-definition Group-Project-CE-Prod:$LATEST_TD_REVISION
        //              '''
        //          }
        //      }
        //  }
    }
}