pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo 'Building'
        sh './gradlew build --no-daemon'
        archiveArtifacts artifacts: 'dist/trainSchedule.zip'
      }
    }
    stage('Build Docker image') {
      when {
        branch 'master'
      }  
      steps {
        script {
          app = docker.build("novemberpain/trainSchedule")
          app.inside {
            sh 'echo $(curl localhost:8000)'
          }
        }
      }
    }
    stage('Push docker image') {
      when {
        branch 'master'
      }
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
          }
        }
      }
    }
  }
}
