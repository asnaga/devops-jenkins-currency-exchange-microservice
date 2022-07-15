pipeline {

  environment {
    dockerimagename = "900832/nodeapp"
    dockerImage = ""
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
		//stage("Compile") {
			//steps {
				//sh "mvn clean compile"
			//}
		//}
		stage("Unit_Test") {
			steps {
				sh "mvn test"
			}
		}
		stage("Build") {
			steps {
				sh "mvn package -DskipTests"
			}
		}
		stage("Docker_Image_Build") {
			steps {
				script {
					dockerImage = docker.build dockerimagename
					echo "Build Success"
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
						//dockerImage.push()
						dockerImage.push('latest')
					}
					
				}
			}
		}
        
		stage('Deploying_to_EKS') { 
		    steps { 
			   script { 
			     kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
	            }
            }
        }			
   } 	
	post {
		always {
			echo "This command runs always"
			 //mail bcc: '', body: 'TEST Sending SUCCESS email from jenkins', cc: '', from: '', replyTo: '', subject: 'SUCCESS BUILDING PROJECT $env.JOB_NAME', to: 'rohith.b@arisglobal.com'
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


