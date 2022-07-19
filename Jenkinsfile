pipeline {

  environment {
    dockerimagename = "/nodeapp"
    dockerImage = ""
    Dev_Emailid = ""
  }

  agent any

  stages {
    stage("Checkout") {
      steps {
        git branch: 'master', url: 'https://github.com/asnaga/devops-jenkins-currency-exchange-microservice.git'
        sh 'mvn --version'
        sh 'docker --version'
        //sh 'node --version'
        echo "Build"
        echo "PATH - $PATH"
        echo "BUILD NUMBER - $env.BUILD_NUMBER"
        echo "BUILD ID - $env.BUILD_ID"
        echo "JOB NAME - $env.JOB_NAME"
        echo "BUILD TAG - $env.BUILD_TAG"
        echo "BUILD URL - $env.BUILD_URL"
      }
    }

  stage("Unit_Test") {
      steps {
        script {
          try {
            echo "Running Unit Test"
            sh "mvn test"
            } catch (e) {
              currentBuild.Result = 'FAILURE'
              throw e
            }

          }
        }
        post {
          failure {
            sh 'return'
            //emailtext attachlog: true, body: 'Unit Test case has failed', subject: 'FAILED', to "${Dev_Emailid}"
          }
        }
      }

  stage("Build") {
        steps {
          sh "mvn package -DskipTests"
        }
        post {
          failure {
            sh 'return'
            //emailtext attachlog: true, body: 'Unit Test case has failed', subject: 'FAILED', to "${Dev_Emailid}"
          }
        }
      }

  stage("Docker_Image_Build") {
        steps {
          script {
            dockerImage = docker.build dockerimagename
            echo "Build Success"
          }
        }
        post {
          failure {
            sh 'return'
            //emailtext attachlog: true, body: 'Unit Test case has failed', subject: 'FAILED', to "${Dev_Emailid}"
          }
        }
      }

  stage("Docker_Image_Push") {
        environment {
          registryCredential = 'dockerhublogin'
        }
        steps {
          script {
            docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
              dockerImage.push('latest')
            }
          }
        }
        post {
          failure {
            sh 'return'
            //emailtext attachlog: true, body: 'Unit Test case has failed', subject: 'FAILED', to "${Dev_Emailid}"
          }
        }
        }


    stage('Deploying_to_EKS') {
        steps {
          script {
            kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
          }
      }
      post {
        failure {
          sh 'return'
          //emailtext attachlog: true, body: 'Unit Test case has failed', subject: 'FAILED', to "${Dev_Emailid}"
        }
      }
    }

  }
  post {
       always {
         echo "This command runs always"
         //mail bcc: '', body: 'TEST Sending SUCCESS email from jenkins', cc: 'saurav.kumar@arisglobal.com', from: 'rohith.b@arisglobal.com', replyTo: '', subject: 'SUCCESS BUILDING PROJECT $env.JOB_NAME', to: 'rohith.b@arisglobal.com'
       }
       success {
         echo "this command executes only when all stages succeed"
         mail bcc: '', body: 'TEST Sending SUCCESS email from jenkins', cc: 'saurav.kumar@arisglobal.com', from: 'rohith.b@arisglobal.com', replyTo: '', subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}", to: 'rohith.b@arisglobal.com'
       }
       failure {
         echo "this command executes when one of the stages failed"
         mail bcc: '', body: 'TEST Sending FAILURE email from jenkins', cc: 'saurav.kumar@arisglobal.com', from: 'rohith.b@arisglobal.com', replyTo: '', subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}", to: 'rohith.b@arisglobal.com'
       }
     }
}
