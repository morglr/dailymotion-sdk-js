pipeline {
  environment {
    DOCKER_HOST = "tcp://inspect-02.adm.dc3.dailymotion.com:4243"
    serviceName = "dailymotion-sdk-js"

  }

  agent {
    label 'sdk-agent'
  }

  stages{
    stage ("Quality") {
      steps {
        parallel(
          "Integrity" : {
            sh 'make integrity'
          }
        )
      }

      post {
        always {
          sh "docker-compose down"
        }
      }
    }

    stage ("Deploy prod") {
      when {
        branch "prod"
      }
      steps {
        sh 'ssh-keyscan prov-04.adm.dc3.dailymotion.com >> ~/.ssh/known_hosts'
        sh 'make'
        sh 'make deploy-prod'
      }

      post {
        success {
          slackSend (color: 'good', channel: "#sdk-js-release", message: "dailymotion-sdk-js pipeline run #${env.BUILD_NUMBER} (<${RUN_DISPLAY_URL}|Open>) has been deployed to Prod")
          wrap([$class: 'BuildUser']) {
            sh 'echo ${BUILD_USER} > build_user_name.txt'
            script {
              def USER_NAME = readFile('./build_user_name.txt').split("\r?\n")
              sh """
                DATE=`echo \$(date '+%Y-%m-%dT%H:%M:%S%z')`
                curl -i -X POST --data-urlencode 'data={"title": "[DAILYMOTION-SDK-JS] ${env.JOB_NAME} release - Build #${env.BUILD_NUMBER}","type":"release","datetime": "'\$DATE'","author": "${USER_NAME}","children": [{"title": "Link to build", "description": "${RUN_DISPLAY_URL}"}]}' http://events.dailymotion.com/api/document
              """
            }
          }
        }
      }
    }
  }

}

