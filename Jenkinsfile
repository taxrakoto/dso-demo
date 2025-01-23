pipeline {
  agent {
    kubernetes {
      cloud 'Ambohitsirohitra'
      yamlFile 'nginx-pod.yaml'
      defaultContainer 'nginx'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('nginx') {
              sh 'echo " tokony eto no m-build " '
            }
          }
        }
      }
    }
    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('nginx') {
              sh 'echo "mvn test" '
            }
          }
        }
      }
    }
    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('nginx') {
              sh 'echo "mvn package -DskipTests"'
            }
          }
        }
      }
    }

    stage('Deploy to Dev') {
      steps {
        // TODO
        sh "echo done"
      }
    }
  }
}
