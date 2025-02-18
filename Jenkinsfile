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
    // stage('Build') {
    //   parallel {
    //     stage('Compile') {
    //       steps {
    //         container('maven') {
    //           sh 'cat nginx-pod.yaml && mvn compile'
    //         }
    //       }
    //     }
    //   }
    // }
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
                  sh 'mvn org.owasp:dependency-check-maven:check -Dformat=XML -DoutputDirectory=$WORKSPACE'
                }
              }
             }
///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// build archive: dependency ckeck report: uncomment to store report as build archive in jenkins server (use storage space)             
            // post {
            //   always {
            //     archiveArtifacts allowEmptyArchive: true, artifacts:'target/dependency-check-report.xml', fingerprint: true, onlyIfSuccessful: true
            //     dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            //    }
            // }
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
          }
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// sonarqube test
        stage ('Sonarqube') {
          environment { scannerHome = tool 'SonarQube-Scanner'}
          steps {
              withSonarQubeEnv ('SonarQube') {
              sh """
                mvn compile
                ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=dso-key
              """
              }
          }
        }
// end of sonar
///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
      }
    }
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
    stage ('Build image in a tarball') {
          agent {label 'docker'}   //specify a different pod to use
          steps {
            container('kaniko') { 
              sh """
              /kaniko/executor \
              -f `pwd`/Dockerfile \
              -c `pwd` --insecure \
              --skip-tls-verify \    
              --tar-path=`pwd`/image.tar \
              --no-push
              """
              }
          }
        }

    stage ('Image Analysis'){
      agent {label 'docker'}
      parallel {
        stage ('lint') {
          steps {
            container ('dockle') { sh 'dockle --input `pwd`/image.tar'}
          }
        }
        stage ('scan') {
          steps { 
            container ('trivy') { sh 'trivy image --input `pwd`/image.tar'}
          }
        }
      }
    }


    stage ('Push image to Registry') {
          agent {label 'docker'}   //specify a different pod to use
          steps {
            container('kaniko') { 
              sh """
              /kaniko/executor \
              -c `pwd` --insecure \
              --skip-tls-verify \
              --cache=true \
              --tar-path=`pwd`/image.tar \
              --destination=docker.io/taxrakoto/dso-demo:latest     
              """
              }
          }
        }


///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
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
        // stage ('Build and Push Image') {
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
