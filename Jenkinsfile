#!/usr/bin/env groovy

def build_image(dirname, tags){
  node('docker'){
    container('docker-runner'){
      deleteDir()
      unstash "source"

      dir("${dirname}"){
        sh "tags=${tags} sh build-images.sh"
        sh "tags=${tags} sh push-images.sh"
      }
    }
  }
}

pipeline {
  agent none
  
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
    timeout(time: 3, unit: 'HOURS')
  }
  
  triggers {
    cron('@daily')
  }
  
  environment {
    DOCKER_REGISTRY_HOST = "${env.DOCKER_REGISTRY_HOST}"
  }

  stages {
    stage('prepare'){
      agent { label 'generic' }
      steps {
        checkout scm
        stash name: "source", includes: "**"
      }
    }
    
    stage('build images'){
      steps {
        parallel(
          "centos6"   : { build_image('rpm', 'centos6') },
          "centos7"   : { build_image('rpm', 'centos7') },
          "rawhide"   : { build_image('rpm', 'rawhide') },
          "ubuntu1604": { build_image('deb', 'ubuntu1604') },
          )
      }
    }
    
    stage('result'){
      steps {
        script { currentBuild.result = 'SUCCESS' }
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