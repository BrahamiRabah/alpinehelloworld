pipeline {
     environment {
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_tag = "latest"
       STAGING = "rabah-staging"
       PRODUCTION = "rabah-prod"
     }
     agent none
     stages {
         stage('build image') {
            agent any
            steps {
              script {
                sh 'docker build -t rabah94/${IMAGE_NAME}:${IMAGE_tag} .'
              }
            }
         }
         stage('run container') {
            agent any
            steps {
              script {
                sh '''
                  docker run -d -p 80:5000 --name ${IMAGE_NAME} -e PORT=5000 rabah94/${IMAGE_NAME}:${IMAGE_tag}
                  sleep 5
                '''
              }
            }
         }
         stage('curl test') {
            agent any
            steps {
              script {
                sh '''
                   curl http://localhost | grep -q "Hello world!"
                '''
              }
            }
         }
         stage('clean container') {
            agent any
            steps {
              script {
                sh '''
                   docker stop ${IMAGE_NAME}
                   docker rm -f ${IMAGE_NAME}
                '''
              }
            }
         }
         stage('push images to production project') {
            when {
                   expression { GIT_BRANCH == 'origin/master' }
                 }
            agent any
            environment {
               HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
              script {
                sh '''
                   heroku container:login
                   heroku create ${PRODUCTION} || echo "project already existe"
                   heroku container:push -a ${PRODUCTION} web
                   heroku container:release -a ${PRODUCTION} web
                '''
              }
            }
         }
         stage('push images to the staging project') {
            when {
                   expression { GIT_BRANCH == 'origin/master' }
                 }
            agent any
            environment {
               HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
              script {
                sh '''
                   heroku container:login
                   heroku create ${STAGING} || echo "project already existe"
                   heroku container:push -a ${STAGING} web
                   heroku container:release -a ${STAGING} web
                '''
              }
            }
         }
     }
     post {
       success {
         slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
         }
      failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
          }  
     }
}
