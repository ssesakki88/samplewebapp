def getDockerTag(){
    def tag = sh script: 'git rev-parse --short HEAD', returnStdout: true
    return tag
  }


pipeline {
    agent any 

    environment{
      docker_tag = getDockerTag()
      docker_auth = credentials('docker-creds')
    }

    stages {

      stage ('execute paralelly'){
        parallel{
          stage('sonarqube analysis'){

            agent {
              docker {
                image 'maven'
                args '-v $HOME/.m2:/root/.m2'
              }
            }  

            steps{
              script{
                withSonarQubeEnv('sonarqube') { 
                    sh "printenv"
                    sh "mvn sonar:sonar"
                }

                timeout(time: 1, unit: 'HOURS') {
                          def qg = waitForQualityGate()
                          if (qg.status != 'OK') {
                              error "Pipeline aborted due to quality gate failure: ${qg.status}"
                  }
                }
              }
            }
          }

          stage('maven build'){

            agent {
              docker {
                image 'maven'
                args '-v $HOME/.m2:/root/.m2'
              }
            }  

            steps {
              script {
                sh "mvn clean install"
              }
            }

          }
        }
      }

    stage("docker build"){
      steps{
        script{
          sh 'cp -r ../java-app-pipeline@2/target .'
          sh "docker build . -t deekshithsn/web-app:$docker_tag"
        }
      }
    }

    stage("docker push stage"){
      steps{
        script{
          sh """
              docker login -u $docker_auth_USR -p $docker_auth_PSW
              docker push deekshithsn/web-app:$docker_tag
          """
          currentBuild.description = "image name: deekshithsn/web-app:$docker_tag"
        }
      }
    }

    }
}