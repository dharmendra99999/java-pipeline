pipeline {
	agent {	
		label 'Ubuntu'
		}
	stages {
		stage("SCM") {
			steps {
				git branch: 'main', url: 'https://github.com/dharmendra99999/java-pipeline.git'
				}
			}

		stage("build") {
			steps {
				sh 'sudo mvn dependency:purge-local-repository'
				sh 'sudo mvn clean package'
				}
			}
		stage("Image") {
			steps {
				sh 'sudo docker build -t java-repo:$BUILD_TAG .'
				sh 'sudo docker tag java-repo:$BUILD_TAG dharmendra/pipeline-java:$BUILD_TAG'
				}
			}
				
	
		stage("Docker Hub") {
			steps {
			withCredentials([string(credentialsId: 'docker_hub_passwd', variable: 'docker_hub_password_var')])   {
				sh 'sudo docker login -u dharmendra99999 -p ${docker_hub_password_var}'
				sh 'sudo docker push dharmendra99999/pipeline-java:$BUILD_TAG'
				}
			}
		}

		stage("QAT Testing") {
			steps {
				sh 'sudo docker run -dit -p 8082:8080 --name web1 dharmendra99999/pipeline-java:$BUILD_TAG'
				}
			}
	 	stage("testing website") {
			steps {
				retry(5) {
				sh "curl --silent http://54.151.131.114:8080/java-web-app/ | grep -i india"
				}
	   		}
		}

		stage("Approval status") {
			steps {
				script {  
                		Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                		echo 'userInput: ' + userInput
				}
	 		}
		}
		
		stage("Prod Env") {
			steps {
			 sshagent(['slave-1']) {
			    sh 'ssh -o StrictHostKeyChecking=no ec2-user@54.169.77.88 sudo docker rm -f $(sudo docker ps -a -q)' 
	                    sh "ssh -o StrictHostKeyChecking=no ec2-user@54.169.77.88 sudo docker run  -d  -p  49153:8080  dharmendra99999/pipeline-java:$BUILD_TAG"
				}
			}
		}
    	}
}
