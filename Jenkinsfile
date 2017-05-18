pipeline {
  agent none
  
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
    timeout(time: 1, unit: 'HOURS')
  }
  
  triggers {
    cron('@daily')
  }

  stages {
    stage('prepare'){
      agent { label 'generic' }
      steps {
        git 'https://github.com/italiangrid/pkg.base'
        stash name: "source", include: "./*"
      }
    }
    
    stage('build images'){
      steps {
        parallel(
          "rpms": {
            node('docker'){
              unstash "source"
              dir('rpm') {
                sh "sh build-images.sh"
                sh "sh push-images.sh"
              }
            }
          },
          "debs": {
            node('docker'){
              unstash "source"
              dir('deb'){
                sh "sh build-images.sh"
                sh "sh push-images.sh"
              }
            }
          }
          )
      }
    }
  }
  
  post {
    failure {
      slackSend color: 'danger', message: "${env.JOB_NAME} - #${env.BUILD_NUMBER} Failure (<${env.BUILD_URL}|Open>)"
    }
    changed {
      script{
        if('SUCCESS'.equals(currentBuild.result)) {
          slackSend color: 'good', message: "${env.JOB_NAME} - #${env.BUILD_NUMBER} Back to normal (<${env.BUILD_URL}|Open>)"
        }
      }
    }
  }
}
