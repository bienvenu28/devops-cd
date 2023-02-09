pipeline {

  agent any

  stages {
    stage('Unit Test') {
      steps {
          nodejs('Node-16.15.1') {
              sh 'echo "### Install packages ####"'
              sh 'yarn --cwd ./react-calculator install'
              sh 'echo "#### Run Unit Test ####"'
              sh 'yarn --cwd ./react-calculator test a --watchAll=false'
         }
      }
    }
    stage('Application build') {
      steps {
          nodejs('Node-16.15.1') {
              sh 'echo "#### Run Build ####"'
              sh 'yarn --cwd ./react-calculator build'
         }
      }
    }
    stage('Pre-production') {
     steps {
        sh 'echo "### Deploying the web app in pre-production###"'
        sh '''
            # we launch the docker run command only if the react-calculator-pre-prod container is not running
            # BUILD_PATH="/home/vagrant/workspace/react-calculator-pipeline"
            if [ ! "$(docker ps | grep -w react-calculator-pre-prod )" ]; then

               docker run --name react-calculator-pre-prod --rm -p 8081:80 \
               -v $(pwd)/react-calculator/build:/usr/share/nginx/html:ro -d nginx

               echo "Web app successfully deployed in pre-production. You may see it on localhost:8081"
            else
               echo "Web app refreshed and already deployed in pre-production. You may see it on localhost:8081"
            fi
          '''
      }
     }
     stage('Production') {
          steps {
             sh 'echo "### Deploying the web app in production###"'
             sh '''
                 # we launch the docker run command only if the react-calculator/prod container is not running
                 # BUILD_PATH="/home/vagrant/workspace/react-calculator-pipeline"
                 # BUILD_PATH="$(pwd)"
                 if [ ! "$(docker ps | grep -w react-calculator-prod )" ]; then

                    docker run --name react-calculator-prod --rm -p 8082:80 --ip=172.17.0.2:80 \
                    -v $(pwd)/react-calculator/build:/usr/share/nginx/html -d nginx

                    echo "Web app successfully deployed in production. You may see it on localhost:8082"
                 else
                    echo "Web app refreshed and already deployed in production. You may see it on localhost:8082"
                 fi
               '''
          }
     }
   stage('Monitoring') {
    steps {
      sh 'echo "### Launching the nginx-prometheus-exporter container $(pwd) ###" '

      sh '''
      [ ! "$(docker ps | grep -w nginx-exporter )" ] && docker run --rm -p 9113:9113 -d \
      --name nginx-exporter nginx/nginx-prometheus-exporter -nginx.scrape-uri http://172.17.0.2:80/metrics \
      || echo "nginx-exporter is already running"
      '''

      sh 'echo "### Launching prometheus and grafana for monitoring ###" '
      sh 'docker-compose -f ./monitoring/docker-compose.yml up -d'
    }
   }
  }
  /*
  post {
    success {
        slackSend  color: "good", channel: "#réalisation-du-projet-devops", message: "Build succeeds - ${env.JOB_NAME} ${env.BUILD_NUMBER}"
    }
    failure {
        slackSend color: "danger", channel: "#réalisation-du-projet-devops", message: "Build fails - ${env.JOB_NAME} ${env.BUILD_NUMBER}"
    }
  }*/
}
