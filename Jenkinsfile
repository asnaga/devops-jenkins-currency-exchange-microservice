pipeline {

  environment {
    dockerimagename = "700080035327.dkr.ecr.us-east-1.amazonaws.com/lscp-mscex-poc-01:${BUILD_NUMBER}"
    AWS_ACCESS_KEY_ID     = credentials('accesskey')
    AWS_SECRET_ACCESS_KEY = credentials('secret_access_key')    
    dockerImage = ""
    Dev_Emailid = ""
    //DevOps = "saurav.kumar@arisglobal.com, rohith.b@arisglobal.com"
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
            emailext attachLog: true, body: "${JOB_NAME} - Unit Test case Failed - ${BUILD_NUMBER}", subject: 'Pipeline Failed', to: "${DevOps}"
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
            emailext attachLog: true, body: "${JOB_NAME} - Build Failed - ${BUILD_NUMBER}", subject: 'Pipeline Failed', to: "${DevOps}"
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
            emailext attachLog: true, body: "${JOB_NAME} - Docker build - ${BUILD_NUMBER}", subject: 'Pipeline Failed', to: "${DevOps}"
          }
        }
      }

  stage("Docker_Image_Push") {
        environment {
          registryCredential = 'dockerhublogin'
        }
        steps {
          //script {
            //docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
              //dockerImage.push('latest')
            //}
            script {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 700080035327.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker push 700080035327.dkr.ecr.us-east-1.amazonaws.com/lscp-mscex-poc-01:${BUILD_NUMBER}'
              }
          }
        post {
          failure {
            sh 'return'
            emailext attachLog: true, body: "${JOB_NAME} - Docker Push Failed - ${BUILD_NUMBER}", subject: 'Pipeline Failed', to: "${DevOps}"
          }
        }
        }

    stage('Deploying_to_EKS') {
        steps {
          script {
            sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 700080035327.dkr.ecr.us-east-1.amazonaws.com'
            kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
          }
      }
      post {
        failure {
          sh 'return'
          emailext attachLog: true, body: "${JOB_NAME} - Deployment Failed - ${BUILD_NUMBER}", subject: 'Pipeline Failed', to: "${DevOps}"
        }
      }
    }

  }
  post {
       always {
         echo "This command runs always"
         //mail bcc: '', body: 'TEST Sending SUCCESS email from jenkins', cc: '', from: '', replyTo: '', subject: 'SUCCESS BUILDING PROJECT $env.JOB_NAME', to: 'rohith.b@arisglobal.com'
         emailext attachLog: true, body: "${JOB_NAME} - Pipeline Execution - ${BUILD_NUMBER}", subject: 'Pipeline Execution Done', to: "${DevOps}"
         //emailext attachLog: true, body: 'Unit Test case has failed', subject: 'FAILED', to "${DevOps}"
       }
       success {
         echo "this command executes only when all stages succeed"
         //mail bcc: '', body: 'TEST Sending SUCCESS email from jenkins', cc: '', from: '', replyTo: '', subject: 'SUCCESS BUILDING PROJECT $env.JOB_NAME', to: 'rohith.b@arisglobal.com'
       }
       failure {
         echo "this command executes when one of the stages failed"
         //mail bcc: '', body: 'TEST Sending FAILURE email from jenkins', cc: '', from: '', replyTo: '', subject: 'ERROR BUILDING PROJECT $env.JOB_NAME', to: 'rohith.b@arisglobal.com'
       }
     }
}
