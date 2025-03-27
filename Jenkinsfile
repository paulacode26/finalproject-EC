pipeline {
    agent any
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

        stage('Build and Push Docker Image'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
                }
            }
            steps{
                sh '''
                    amazon-linux-extras install docker
                    docker build -t my-docker-image .
                    withCredentials([
                    usernamePassword(
                    credentialsId: 'Group-Project-CE', 
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY', 
                    usernameVariable: 'AWS_ACCESS_KEY_ID')
                    ]) {
                        sh '''
                            aws --version
                            

                        
                        '''
                    }
                '''
            }
        } 

        stage('Deploy to AWS') {
             agent {
                 docker {
                     image 'amazon/aws-cli'
                     reuseNode true
                     args '-u root --entrypoint=""'
                 }
            }
           
             steps {
                withCredentials([usernamePassword(credentialsId: 'my-s3-key', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')])
                {  
                     sh '''
                         aws --version
                        yum install jq -y
                       
                         LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                         aws ecs update-service --cluster my-new-Cluster-Prod --service my-new-Service-Prod --task-definition my-new-TaskDefinition-Prod:$LATEST_TD_REVISION
                     '''
                 }
             }
         }
    }
}