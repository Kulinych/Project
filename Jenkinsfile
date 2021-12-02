pipeline { 
    environment { 
    registry = "kulinych/md-sa2-18-21" 
    registryCredential = 'docker'
    dockerImage = 'md-sa2-18-21'
    }
    agent {
        node {
            label 'master'  
        }
    }
    stages{
    stage('clone app'){
      steps{
        git url: 'https://github.com/Kulinych/project.git', branch: 'master'
      }    
    }  
    stage('Show files'){
      steps{
          sh"""
          ls -l 
          """
        } 
    }  
    stage('build container ') {
      steps {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        } 
      }
    }
    stage('Unit test') {
      steps {
        sh """
        docker run -t "$registry:$BUILD_NUMBER" manage.py test -v 2
        """
        }
      }
    stage('push build') {
      steps {
        script {
        docker.withRegistry( '', registryCredential ) {
        dockerImage.push()
        }
      }
    }
    stage('install Helm chart'){
      steps{ 
          sh"""
          helm upgrade --install --atomic md-sa2-18-21 --namespace default ./helm-chart --values ./helm-chart/values.yaml --set image.tag=$BUILD_NUMBER  
        """
        }    
    }
}
post {
    always {
            deleteDir()
        }
        success {
            slackSend (color: '#00FF00', message: "SUCCESSFULL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }