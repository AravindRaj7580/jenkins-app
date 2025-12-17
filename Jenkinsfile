pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'a6f6fa8e-131d-4c56-ab81-5870f68cfc4c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Docker') {
            sh 'docker build -t my-playwiright .'
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
                    echo "Hello World"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(2) {
                    input 'i\'m alowing to deploy the application!'
                }
            }
        }

        stage('Deploy - staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install netlify-cli@20.1.1 node-jq
                   node_modules/.bin/netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy \
                      --dir=build \
                      --site=$NETLIFY_SITE_ID \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --json > deploy-output.json
                   node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                '''
            }
        }

        stage('Deploy - prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install netlify-cli@20.1.1
                   node_modules/.bin/netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy \
                      --dir=build \
                      --prod \
                      --site=$NETLIFY_SITE_ID \
                      --auth=$NETLIFY_AUTH_TOKEN
                '''
            }
        }
    }
    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}