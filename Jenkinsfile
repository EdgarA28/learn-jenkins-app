pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '35d32d05-e100-42be-8528-f6df5c0da6c8'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Tests'){
            parallel{
                    stage('Unit tests'){
                        agent{
                            docker{
                                image 'node:18-alpine'
                                reuseNode true
                            }
                        }
                        steps {
                            sh '''
                            #test -f build/index.html
                            npm test
                            '''
                        }
                    }
                    
                    stage('E2E'){
                        agent {
                            docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '-u root:root'
                                }
                        }
                        steps{
                            sh '''
                                npm install serve
                                node_modules/.bin/serve -s build & 
                                sleep 10
                                npx playwright test --reporter=html
                            '''
                        }
                        post {
                        always{
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version   
                    echo "Deploying to staging Site ID: $NETLIFY_SITE_ID "
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }
        stage('Approval'){
            steps{
                timeout(time: 25, unit: 'SECONDS') {
                    input message: 'Ready to deploy', ok: 'yes, i am sure'
                }
                
            }

        }
        stage('Deploy prod') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version   
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID "
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod E2E'){
            agent {
                docker{
                image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                reuseNode true
                args '-u root:root'
                    }
            }
            environment{
            CI_ENVIRONMENT_URL = 'https://melodic-kitsune-a5af75.netlify.app'
        }

            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
            always{
                junit 'jest-results/junit.xml'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML E2E Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }
    }

    }



}