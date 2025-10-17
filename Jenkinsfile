pipeline {
    agent any

    environment{
        NETLYFY_SIDE_ID = '35d32d05-e100-42be-8528-f6df5c0da6c8'
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
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('E2E'){
            agent {
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install -g serve
                    serve -s build
                    npx playwright test
                '''
            }
        }
        // stage('Deploy') {
        //     agent {
        //         docker{
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             npm install netlify-cli
        //             node_modules/.bin/netlify --version
        //             echo "Deploy to production side ID: $NETLIFY_SIDE_ID"
        //         '''
        //     }
        // }
    }
    post {
        // sucess{
        //     archiveArtifacts artifacts: 'build/**'
        // }
        always{
            cleanWs()
        }
    }


}