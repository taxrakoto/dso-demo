pipeline {
  agent {
    kubernetes {
      cloud 'Ambohitsirohitra'
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'cat nginx-pod.yaml && mvn compile'
            }
          }
        }
      }
    }
    stage('Static Analysis') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
        stage ('SCA'){
          steps {
            container ('maven'){
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE'){
                  sh 'mvn org.owasp:dependency-check-maven:check -DoutputDirectory=$WORKSPACE'
                }
              }
             }
            post {
              always {
                archiveArtifacts allowEmptyArchive: true, artifacts:'target/dependency-check-report.html', fingerprint: true, onlyIfSuccessful: true
                dependencyCheckPublisher pattern: 'target/dependency-check-report.html'
               }
            }
          }
      }
    }
    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }
//  build and publish to dockerhub phase
        // stage ('OCI Image Bnp') {
        //   agent {label 'docker'}   //specify a different pod to use
        //   steps {
        //     container('kaniko') { sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/taxrakoto/dso-demo'}
        //   }
        // }
//  build and publish to dockerhub phase        
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
