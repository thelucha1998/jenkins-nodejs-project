pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    // REGISTRY = 'gitlab-jenkins.opes.com.vn'
    // the project name
    // make sure your robot account have enough access to the project
    // HARBOR_NAMESPACE = 'jenkins-harbor'
    // docker image name
    // APP_NAME = 'docker-example'
    // ‘robot-test’ is the credential ID you created on the KubeSphere console
    // HARBOR_CREDENTIAL = credentials('harbor')
  }
  stages {
    stage('Login') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        // sh '''echo $HARBOR_CREDENTIAL_PSW | docker login $REGISTRY -u 'admin' --password-stdin'''
      }
    }
    stage('Build') {
      steps {
        sh 'docker build -t eden266/nodejs-project:v2 .'
	sh 'cd prod && docker build -t eden266/nodejs-project-prod:v2 .'
        // sh 'docker build -t $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME:jenkins-nodejs-project .'
      }
    }

    stage('Push') {
      environment {
        registryCredential = 'dockerhub'
      }
      steps {
        sh 'docker push eden266/nodejs-project:v2'
        sh 'docker pull eden266/nodejs-project:v2'
        // sh 'docker push  $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME:jenkins-nodejs'
        /*
        script {
          
          docker.withRegistry( 'https://gitlab-jenkins.opes.com.vn', registryCredential ) {
            sh 'docker push  $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME:jenkins-nodejs'
          }
        }
        */
      }
    }
      stage('Deploy Dev') {
        steps{
          script {
            sshagent(credentials : ['my-ssh-key']) {
                sh 'ssh -o StrictHostKeyChecking=no -i my-ssh-key opes@10.0.10.2 "hostname && cd jenkins-nodejs-project && helm upgrade --install jenkins-nodejs-dev ./node-app-chart"'
                
            }
          }
        }     
      }
      stage('Deploy Prod') {
        steps{
          script {
            sshagent(credentials : ['my-ssh-key']) {
                sh 'ssh -o StrictHostKeyChecking=no -i my-ssh-key opes@10.0.10.2 "hostname && cd jenkins-nodejs-project/prod && helm upgrade --install jenkins-nodejs-prod ./node-app-chart"'
                
            }
          }
        }     
      }
      
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}
