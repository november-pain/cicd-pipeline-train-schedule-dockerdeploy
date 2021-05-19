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
          app = docker.build("novemberpain/train-schedule")
          app.inside {
            sh 'echo $(curl localhost:8080)'
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
    stage('Deploy docker container in production') {
      when {
        branch 'master'
      }
      steps {
        input 'Are you sure?'
        milestone(1)
        withCredentials ([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
          script {
            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker pull novemberpain/train-schedule:${env.BUILD_NUMBER}\""
            try {
              sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker stop train-schedule\""
              sh "sshpass -p '$USERNAME' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker rm train-schedule\""
            } catch (err) {
                echo: 'caught error: $err'
            }
            sh "sshpass -p 'USERNAME' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod.ip} \"docker run --restart always --name train-schedule -p 8080:8080 -d novemberpain/train-schedule:${env.BUILD_NUMBER}\""
          }
        }
      }
    }
  }
}
