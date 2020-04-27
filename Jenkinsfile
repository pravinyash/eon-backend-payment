pipeline {
   agent any
   environment {
       registry = "magalixcorp/k8scicd"
       
   }
   stages {   
   
   stage('Init'){
       steps{
           //checkout scm;
       script{
       env.BASE_DIR = pwd()
       env.CURRENT_BRANCH = env.BRANCH_NAME
       env.IMAGE_TAG = getImageTag(env.CURRENT_BRANCH)
       env.APP_NAME= getEnvVar('APP_NAME')
       env.IMAGE_NAME = "${APP_NAME}-app"
       }
       }
   }
      
   stage('Cleanup'){
       steps{
           sh '''
           docker rmi $(docker images -f 'dangling=true' -q) || true
           docker rmi $(docker images | sed 1,2d | awk '{print $3}') || true
           '''
       }
   }
    stage('Build Dockerimage'){
       steps{
           sh '''
            docker build -t ${DOCKER_REGISTRY_URL}/${DOCKER_PROJECT_NAMESPACE}/${IMAGE_NAME}:${RELEASE_TAG} --build-arg APP_NAME=${IMAGE_NAME}  -f app/Dockerfile app/.
           '''
       }
   }
    stage('push Dockerimage'){      
     steps{
      sh '''
      echo $DOCKER_PASSWD | docker login --username ${DOCKER_USERNAME} --password-stdin ${DOCKER_REGISTRY_URL} 
      docker push ${DOCKER_REGISTRY_URL}/${DOCKER_PROJECT_NAMESPACE}/${IMAGE_NAME}:${RELEASE_TAG}
      docker logout
      '''
     }
    }   
  
   stage('Deploy'){
       steps{
       withCredentials([file(credentialsId: "${JENKINS_GCLOUD_CRED_ID}", variable: 'JENKINSGCLOUDCREDENTIAL')])
       {
       sh """
           gcloud auth activate-service-account --key-file=${JENKINSGCLOUDCREDENTIAL}
           gcloud config set compute/zone asia-southeast1-a
           gcloud config set compute/region asia-southeast1
           gcloud config set project ${GCLOUD_PROJECT_ID}
           gcloud container clusters get-credentials ${GCLOUD_K8S_CLUSTER_NAME}
           chmod +x $BASE_DIR/k8s/process_files.sh
           cd $BASE_DIR/k8s/
           ./process_files.sh "$GCLOUD_PROJECT_ID" "${IMAGE_NAME}" "${DOCKER_PROJECT_NAMESPACE}/${IMAGE_NAME}:${RELEASE_TAG}" "./${IMAGE_NAME}/"
           cd $BASE_DIR/k8s/${IMAGE_NAME}/.
           kubectl apply -f $BASE_DIR/k8s/${IMAGE_NAME}/
           gcloud auth revoke --all
           """
       }
       }
      }      

   }
}
