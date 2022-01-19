pipeline {
  agent any 
  triggers {
        pollSCM(env.GIT_BRANCH == 'main' ? '* * * * *' : env.GIT_BRANCH == 'staging' ? '* * * * *' : '')
    }
  stages {
      stage('Checkout SCM') {
        steps{
          checkout scm
          sh "ls"
          sh "git --version"
          echo "Deployment TO ${env.GIT_BRANCH}"
          script {   env.DOCKER_REGISTRY = 'ibnuzamra'
                     env.DOCKER_IMAGE_NAME = 'frontend'
                     //#Change env DOCKER_IMAGE_APPS
                     env.DOCKER_IMAGE_APPS = 'frontend'
          }
        }
      }
      stage('Build Docker Image') {
        steps{
          script {
            if ( env.GIT_BRANCH == 'staging' ){
              sh "docker image build . -t $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME-stg:${BUILD_NUMBER}"
              sh "docker tag $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME-stg:${BUILD_NUMBER} $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME-stg:${BUILD_NUMBER}"
              echo "Docker Image ${BUILD_NUMBER} Build For Server Staging ${currentBuild.currentResult}"
            }  
            else if ( env.GIT_BRANCH == 'main' ){
              sh "docker image build . -t $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME-prd:${BUILD_NUMBER}"
              sh "docker tag $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME-prd:${BUILD_NUMBER} $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME-prd:${BUILD_NUMBER}"
              echo "Docker Image ${BUILD_NUMBER} Build For Server Production ${currentBuild.currentResult}"
            }
          }  
        }
      }
    
      stage('Push Docker Image') {
            steps {
                script {
                 withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u ibnuzamra -p ${dockerhubpwd}'
                 }
                 sh "docker push $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME-stg:${BUILD_NUMBER}"
                }
            }
      }
    
      stage('Docker Image Delete'){
        steps{
          script {
            if ( env.GIT_BRANCH == 'staging' ){
              sh "docker image rm -f $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME-stg:${BUILD_NUMBER}"
              echo "Docker Image ${BUILD_NUMBER} Delete For Server Staging ${currentBuild.currentResult}"
            }
            else if ( env.GIT_BRANCH == 'main' ){
              sh "docker image rm -f $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME-prd:${BUILD_NUMBER}"
              echo "Docker Image ${BUILD_NUMBER} Delete For Server Production ${currentBuild.currentResult}"
            }
          }  
        }
      }
    
      stage('Deploy TO K8S'){
        steps{
          script {
            if ( env.GIT_BRANCH == 'staging' ){
              //#Change url wget
              sh 'wget https://raw.githubusercontent.com/ibnuzamra/frontend-bp/main/s-frontend-deployment.yml'
              sh 'sed -i "s/versi/$BUILD_NUMBER/g" s-"${DOCKER_IMAGE_APPS}"-deployment.yml'
              sh 'export KOPS_STATE_STORE=s3://k8s-knsrvtf'
              sh 'kops export kubecfg --admin'
              sh 'kubectl apply -f s-"${DOCKER_IMAGE_APPS}"-deployment.yml'
              sh 'rm -rf *'
              echo "Deploy ${BUILD_NUMBER} To Server Staging ${currentBuild.currentResult}"
            }
            else if ( env.GIT_BRANCH == 'main' ){
              //#Change url wget
              sh 'wget https://raw.githubusercontent.com/ibnuzamra/frontend-bp/main/p-frontend-deployment.yml'
              sh 'sed -i "s/versi/$BUILD_NUMBER/g" p-"${DOCKER_IMAGE_APPS}"-deployment.yml'
              sh 'export KOPS_STATE_STORE=s3://k8s-knsrvtf'
              sh 'kops export kubecfg --admin'
              sh 'kubectl apply -f p-"${DOCKER_IMAGE_APPS}"-deployment.yml'
              sh 'rm -rf *'
              echo "Deploy ${BUILD_NUMBER} To Server Production ${currentBuild.currentResult}"
            }
          }  
        }
      }
    }
  post {
        always {
          script {
            if ( env.GIT_BRANCH == 'staging' ){
              echo "DEPLOY NUMBER ${BUILD_NUMBER} TO SERVER STAGING ${currentBuild.currentResult}"
  //            slackSend message: "DEPLOY ${DOCKER_IMAGE_APPS} NUMBER ${BUILD_NUMBER} TO SERVER STAGING ${currentBuild.currentResult}"
  
            }
            else if ( env.GIT_BRANCH == 'main' ){
              echo "DEPLOY NUMBER ${BUILD_NUMBER} TO SERVER STAGING ${currentBuild.currentResult}"
  //            slackSend message: "DEPLOY ${DOCKER_IMAGE_APPS} NUMBER ${BUILD_NUMBER} TO SERVER PRODUCTION ${currentBuild.currentResult}"
            }
          }  
        }
  }  
}
