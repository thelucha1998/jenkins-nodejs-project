pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    REGISTRY = 'gitlab-jenkins.opes.com.vn'
    // the project name
    // make sure your robot account have enough access to the project
    HARBOR_NAMESPACE = 'jenkins-harbor'
    // docker image name
    APP_NAME_DEV = 'docker-jenkins-dev'
    APP_NAME_PROD = 'docker-jenkins-prod'
    // ‘robot-test’ is the credential ID you created on the KubeSphere console
    HARBOR_CREDENTIAL = credentials('harbor')
    IMAGE_TAG_DEV = 'v2dev'
    IMAGE_TAG_PROD = 'v2prod'
  }
  stages {
    stage('Login') {
      steps {
        // sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        sh '''echo $HARBOR_CREDENTIAL_PSW | docker login $REGISTRY -u 'admin' --password-stdin'''
      }
    }

 stage("Code Checkout from GitHub") {
  steps {
   git branch: 'main',
    credentialsId: 'github_access_token',
    url: 'https://github.com/thelucha1998/jenkins-nodejs-project.git'
  }
 }
  stage('Code Quality Check via SonarQube') {

    steps {

     script {

     def scannerHome = tool 'sonarqube';

       withSonarQubeEnv("sonarqube-container") {

       sh "${tool("sonarqube")}/bin/sonar-scanner \
       -Dsonar.projectKey=test-node-js \
       -Dsonar.sources=. \
       -Dsonar.css.node=. \
       -Dsonar.host.url=http://10.0.6.80:9000 \
       -Dsonar.login=squ_3f3d4d90dee367e1b871d7a09ef95f3dba9dd877"

           }

          }

       }

  }
    stage('Build') {
      steps {
        // sh 'docker build -t eden266/nodejs-project:v2 .'
	// sh 'cd prod && docker build -t eden266/nodejs-project-prod:v2 .'
        sh 'docker build -t $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME_DEV:$IMAGE_TAG_DEV .'
	sh 'cd prod && docker build -t $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME_PROD:$IMAGE_TAG_PROD .'
      }
    }

    stage('Push') {
      environment {
        registryCredential = 'harbor'
      }
      steps {
 	// sh 'docker push eden266/nodejs-project:v2'
 	// sh 'docker pull eden266/nodejs-project:v2'
 	// sh 'docker push eden266/nodejs-project-prod:v2'
	// sh 'docker pull eden266/nodejs-project-prod:v2'
	// sh 'docker push  $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME:jenkins-nodejs'
        
        script {
          
          docker.withRegistry( 'https://gitlab-jenkins.opes.com.vn', registryCredential ) {
	    sh 'docker tag $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME_DEV:$IMAGE_TAG_DEV $REGISTRY'
            sh 'docker push  $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME_DEV:$IMAGE_TAG_DEV'
	    sh 'docker tag $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME_PROD:$IMAGE_TAG_PROD $REGISTRY'
	    sh 'docker push  $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME_PROD:$IMAGE_TAG_PROD'
          }
        }
        
      }
    }
      stage('Deploy Dev') {
        steps{
          script {
            sshagent(credentials : ['my-ssh-key']) {
                sh 'ssh -o StrictHostKeyChecking=no -i my-ssh-key opes@10.0.10.2 "docker pull $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME_DEV:$IMAGE_TAG_DEV && cd jenkins-nodejs-project && helm upgrade --install jenkins-nodejs-dev ./node-app-chart"'
                
            }
          }
        }     
      }
      stage('Deploy Prod') {
        steps{
          script {
            sshagent(credentials : ['my-ssh-key']) {
                sh 'ssh -o StrictHostKeyChecking=no -i my-ssh-key opes@10.0.10.2 "docker pull $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME_PROD:$IMAGE_TAG_PROD && cd jenkins-nodejs-project/prod && helm upgrade --install jenkins-nodejs-prod ./node-app-chart"'
                
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
