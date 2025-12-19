pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-southeast-2'
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

        stage('Buid docker image') {
            agent {
                docker {
                    image 'docker:26-cli'
                    reuseNode true
                    args "-v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }

            steps {
                sh '''
                    docker --version
                    docker build -t myjenkinsapp .
                '''
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: '2f9e57d8-8788-49e8-8bcd-4b0a0ccee04b', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                        aws ecs update-service --cluster learn-jenkins-191220251444 --service learn-jenkins-app-service-prod --task-definition learnJenkins-TaskDefinition-prod:3
                        aws ecs wait services-stable --cluster learn-jenkins-191220251444 --services learn-jenkins-app-service-prod
                    '''
                }
                
            }
        }
        
    }
}