pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonartoken') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
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
       
       stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass_nexus', variable: 'docker_password')]) {
                             sh '''
                                docker build -t 20.102.101.194:8083/springapp:${VERSION} .
                                docker login -u admin -p $docker_password 20.102.101.194:8083 
                                docker push  20.102.101.194:8083/springapp:${VERSION}
                                docker rmi 20.102.101.194:8083/springapp:${VERSION}
                            '''
                    }
                }
            }
       }
        stage('indentifying misconfigs using datree in helm charts'){
            steps{
                script{

                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=3329f3dc-4f28-4ead-bf6e-95fb6943c324']) {
                              sh 'helm datree test myapp/'
                        }
                    }
                }
            }
        }
   }
       post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "rohithrajdevops@gmail.com";
		 }
	  }

}
