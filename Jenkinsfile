pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'a6f6fa8e-131d-4c56-ab81-5870f68cfc4c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps {
                sh '''
                    aws --version
                '''
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
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   netlify status
                   netlify deploy \
                      --dir=build \
                      --site=$NETLIFY_SITE_ID \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --json > deploy-output.json
                   node-jq -r '.deploy_url' deploy-output.json
                '''
            }
        }

        stage('Deploy - prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   netlify status
                   netlify deploy \
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