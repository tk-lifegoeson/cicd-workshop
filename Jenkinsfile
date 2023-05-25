pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh "docker build -t lifegoeson/podinfo:${env.BUILD_NUMBER} ."
      }
    }
    stage('Test'){
       steps {
        sh "echo 'Test'"
      }
    }
    stage('Security scan'){
      steps {
        withCredentials([string(credentialsId: 'snyk-api-token', variable: 'snykToken')]){
        sh "snyk auth ${snykToken}"
        sh "snyk test"
        sh "snyk monitor --all-projects"
        }
      }
    }
    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
          sh "docker push supaket/podinfo:${env.BUILD_NUMBER}"
        }
      }
    }
    stage('Docker Remove Image') {
      steps {
        sh "docker rmi supaket/podinfo:${env.BUILD_NUMBER}"
      }
    }
    stage('Deploy') {
      steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
          sh 'cat deployment.yaml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | kubectl apply -f -'
          sh 'kubectl apply -f service.yaml'
        }
      }
  }
}
post {
    success {
      slackSend(message: "Pipeline is successfully completed.")
    }
    failure {
      slackSend(message: "Pipeline failed. Please check the logs.")
    }
}
}