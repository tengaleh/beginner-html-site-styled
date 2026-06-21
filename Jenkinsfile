pipeline {

 agent { label 'k8s-agent' }

 environment {

   IMAGE = "dockerhubuser/website:${BUILD_NUMBER}"
 }

 stages {

   stage('Clone') {

     steps {
          git branch: 'main',
            url: 'https://github.com/tengaleh/beginner-html-site-styled.git'
     }
   }

   stage('Build Image') {

     steps {
       sh 'docker build -t $IMAGE .'
     }
   }

   stage('Docker Login') {

     steps {

       withCredentials([
         usernamePassword(
         credentialsId: 'dockerhub',
         usernameVariable: 'USER',
         passwordVariable: 'PASS')
       ]) {

       sh '''
       echo $PASS | docker login -u $USER --password-stdin
       '''
       }
     }
   }

   stage('Push Image') {

     steps {

       sh '''
       docker push $IMAGE
       docker tag $IMAGE dockerhubuser/website:latest
       docker push dockerhubuser/website:latest
       '''
     }
   }

   stage('Deploy') {

     steps {

       sh '''
       kubectl apply -f deployment.yaml
       kubectl apply -f service.yaml
       kubectl rollout restart deployment website
       '''
     }
   }
 }
}
