Library('momo-shared-Library')
pipeline {
 environment{
  IMAGE_NAME = 'webapp'
  IMAGE_TAG = 'v1'
  DOCKER_PASSWORD = credentials('docker-password')
  HOST_PORT = 80
  CONTAINER_PORT = 80
  IP_DOCKER = '172.17.0.1'
  }
  agent any
  stages {
     stage ('Build'){
         steps {
             scripts{
                 sh '''
                    docker build --no-cache -t $IMAGE_NAME:$IMAGE_TAG .
                 '''
             }
         }
     }

     stage ('Test'){
         steps {
             scripts{
               docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME
               sleep 5
               curl -I http://$IP_DOCKER
               docker stop $IMAGE_NAME
             }
         }
     }

     stage ('Release'){
         steps {
             scripts{
              sh '''
               docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
               echo $DOCKER_PASSWORD| docker login -u $DOCKER_USERNAME --password-stdin
               docker push $OCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
             '''
             }
         }
     }

     stage ('Deploy Review'){
         environment{
            SERVER_IP = 34.202.229.221
            SERVER_USERNAME = ubuntu 
         }
         steps {
             scripts{
                 timeout(time: 30, unit:"MINUTES"){
                     input message: "deploiement sur review ?",ok: 'Yes'
                 } 
                 sshagent(['key-pair']){
                    sh '''
                        echo $DOCKER_PASSWORD| docker login -u $DOCKER_USERNAME --password-stdin
                        ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP docker rm -f $IMAGE_NAME || echo 'All deleted'           
                        ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo"Image Downloaded Succesfully"
                        sleep 30
                        ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                        sleep 5
                        curl -I http://$SERVER_IP:$HOST
                    ''' 
              }
         }
     }
 }
     stage ('Deploy Staging'){
         environment{
            SERVER_IP = 3.80.41.219
            SERVER_USERNAME = ubuntu
         }
         steps {
             scripts{
              timeout(time: 30, unit:"MINUTES"){
                     input message: "deploiement sur review ?",ok: 'Yes'
              } 
               sshagent(['key-pair']){
                    sh '''
                        echo $DOCKER_PASSWORD| docker login -u $DOCKER_USERNAME --password-stdin
                        ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP docker rm -f $IMAGE_NAME || echo 'All deleted'
                        ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo"Image Downloaded Succesfully"
                        sleep 30
                        ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                        sleep 5
                        curl -I http://$SERVER_IP:$HOST
                    '''
             }
         }
    }
}
     stage ('Deploy Prod'){
         #when {expression{GIT_BRANCH == 'origin/prod'}}
         environment{
            SERVER_IP = 34.201.150.231
            SERVER_USERNAME = ubuntu
         }
         steps {
             scripts{
              timeout(time: 30, unit:"MINUTES"){
                     input message: "deploiement sur review ?",ok: 'Yes'
              } 
              sshagent(['key-pair']){
                    sh '''
                        echo $DOCKER_PASSWORD| docker login -u $DOCKER_USERNAME --password-stdin
                        ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP docker rm -f $IMAGE_NAME || echo 'All deleted'
                        ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo"Image Downloaded Succesfully"
                        sleep 30
                        ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                        sleep 5
                        curl -I http://$SERVER_IP:$HOST
                    '''
             }
         }
    }
 }
     post{
         always{
           script
           {
              slackNotifier currentBuild.result
           }
         }
     }     
 }

