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
                  docker run -d -p 8081:80 --name ${IMAGE_NAME} -e PORT=8081 rabah94/${IMAGE_NAME}:${IMAGE_tag}
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
         stage('push images') {
            when {
               expression { GIT_BRANCHE == 'origin/master' }
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
         stage('push images staging') {
            when {
               expression { GIT_BRANCHE == 'origin/master' }
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
}
