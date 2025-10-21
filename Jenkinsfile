pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '35d32d05-e100-42be-8528-f6df5c0da6c8'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME ='myjenkinsapp'
        AWS_DOCKER_REGISTRY= '590980992845.dkr.ecr.us-east-1.amazonaws.com'

    }

    stages {
        // stage('Build') {
        //     agent {
        //         docker{
        //             image 'node:18-alpine'
        //             reuseNode true
        //             args '-u root:root'
        //         }
        //     }
        //     steps {
        //         sh '''
        //             ls -la
        //             node --version
        //             npm --version
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }
        stage('Build Docker image'){
            agent{
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    docker build -t $APP_NAME:$REACT_APP_VERSION .
                    aws ecr get-login-password --region us-east-1 | \
                    docker login\
                    --username AWS \
                    --password-stdin $AWS_DOCKER_REGISTRY
                     docker tag $APP_NAME:$REACT_APP_VERSION \
                     $AWS_DOCKER_REGISTRY/learnjenkinsapp:$REACT_APP_VERSION
                     docker push $AWS_DOCKER_REGISTRY/learnjenkinsapp:$REACT_APP_VERSION
                '''
                }
            }
        }
        stage('Deploy AWS'){
            agent{
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            environment{
                AWS_S3_BUCKET = 'learn-jenkins-bucket-28071993'
                AWS_DEFAULT_REGION ='us-east-1'
                AWS_ECS_CLUSTER = 'JenkinsApp-Cluster-Prod'
                AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
                AWS_ECS_TD = 'LearnJenkinsApp-TaskDefinition-Prod'
            }
            steps{
                    withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh ''' 
                     aws --version
                     sed -i "s#APP_VERSION#/$REACT_APP_VERSION/g" aws/task-definition-prod.json
                     #aws s3 sync build s3://$AWS_S3_BUCKET
                     LATEST_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                     echo $LATEST_REVISION
                     aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD:$LATEST_REVISION
                     #aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD 
                    '''
                }
           
            }
        }
    //     stage('Tests'){
    //         parallel{
    //                 stage('Unit tests'){
    //                     agent{
    //                         docker{
    //                             image 'node:18-alpine'
    //                             reuseNode true
    //                         }
    //                     }
    //                     steps {
    //                         sh '''
    //                         #test -f build/index.html
    //                         npm test
    //                         '''
    //                     }
    //                 }
                    
    //                 stage('E2E'){
    //                     agent {
    //                         docker{
    //                         image 'my-playwright'
    //                         reuseNode true
    //                         args '-u root:root'
    //                             }
    //                     }
    //                     steps{
    //                         sh '''
              
    //                             serve -s build & 
    //                             sleep 10
    //                             npx playwright test --reporter=html
    //                         '''
    //                     }
    //                     post {
    //                     always{
    //                         junit 'jest-results/junit.xml'
    //                         publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report local', reportTitles: '', useWrapperFileDirectly: true])
    //                     }
    //                 }
    //             }
    //         }
    //     }


    //     stage('Deploy Staging'){
    //         agent {
    //             docker{
    //             image 'my-playwright'
    //             reuseNode true
    //             args '-u root:root'
    //             }
    //         }
    //         environment{
    //         CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
    //          }

    //         steps{
    //             sh '''
    //                 netlify --version   
    //                 echo "Deploying to staging Site ID: $NETLIFY_SITE_ID "
    //                 netlify status
    //                 netlify deploy --dir=build --json > deploy-output.json
    //                 CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
    //                 npx playwright test --reporter=html
    //             '''
    //         }
    //         post {
    //         always{
    //             junit 'jest-results/junit.xml'
    //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
    //         }
    //     }
    // }



    //     // stage('Approval'){
    //     //     steps{
    //     //         timeout(time: 25, unit: 'SECONDS') {
    //     //             input message: 'Ready to deploy', ok: 'yes, i am sure'
    //     //         }
                
    //     //     }

    //     // }

    //     stage('Deploy Prod'){
    //         agent {
    //             docker{
    //             image 'my-playwright'
    //             reuseNode true
    //             args '-u root:root'
    //             }
    //         }
    //         environment{
    //         CI_ENVIRONMENT_URL = 'https://melodic-kitsune-a5af75.netlify.app'
    //         }

    //         steps{
    //             sh '''

    //                 netlify --version   
    //                 echo "Deploying to production. Site ID: $NETLIFY_SITE_ID "
    //                 netlify status
    //                 netlify deploy --dir=build --prod
    //                 npx playwright test --reporter=html
    //             '''
    //         }
    //         post {
    //         always{
    //             junit 'jest-results/junit.xml'
    //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML E2E Report', reportTitles: '', useWrapperFileDirectly: true])
    //         }
    //     }
    // }

     }



}