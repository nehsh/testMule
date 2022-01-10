pipeline {
	agent any
		tools{
			maven 'maven-3.8.4'
	    }
		environment
		{ 
          _JAVA_OPTIONS="-Xmx2048M -XX:MaxPermSize=1024m"
          SONAR_RUNNER_OPTS="-Xms2048m -Xmx2048m -XX:MaxPermSize=2048m"
          NEXUS_VERSION = "nexus3"
          NEXUS_PROTOCOL = "http"      
        }		
	    stages {
			stage('Code Checkout') {
				steps {
				checkout scm
				}
			}	   
			stage('Munit Test') {
				steps {
					script {
						configFileProvider([configFile(fileId: 'b16cd1b3-6027-4042-af76-104f3e1f418e', variable: 'Maven_Settings')]) {
							echo "Munit Execution Begins"
							sh "mvn -s $Maven_Settings -Druntime.key=123 -DbuildNumber=${env.BUILD_NUMBER} -Denv=dev clean test" 
							echo "Munit completed"
							archive "target/munit-reports/coverage/**/*"
					   } 
				   }
			   }
			}		
			stage('Upload To Nexus') {
				steps {
					script {
						configFileProvider([configFile(fileId: 'b16cd1b3-6027-4042-af76-104f3e1f418e', variable: 'Maven_Settings')]) {
							echo "Build project/Uploading to nexus Begins"
							sh "mvn -s $Maven_Settings clean package deploy -DbuildNumber=${env.BUILD_NUMBER} -Denv=dev -DskipMunitTests" 
							echo "Build project/Uploading to nexus completed"
					   } 
				   }
			   }
			}				
			stage('Build and Deploy') {
				steps {
					script {
						configFileProvider([configFile(fileId: 'b16cd1b3-6027-4042-af76-104f3e1f418e', variable: 'Maven_Settings')]) {
							echo "Deploying to Mule"
							echo 'Building project'
					        pom = readMavenPom(file: 'pom.xml')		
					        echo "getArtifactId - ${pom.getArtifactId()}"					
							withCredentials([usernamePassword(credentialsId: 'anypoint-platform', usernameVariable: 'DEVOPSUSERNAME', passwordVariable: 'DEVOPSPASSWORD')]) {
								sh "echo ${DEVOPSUSERNAME}"							
								sh 'mvn -s $Maven_Settings mule:deploy -Dmule.artifact=${WORKSPACE}/target/${pom.getArtifactId()}-1.0.0-${BUILD_NUMBER}-mule-application.jar -DskipMunitTests -Dcloudhub.muleVersion=${DEVOPS_CLOUDHUB_MULEVERSION} -Dcloudhub.applicationName=testMule-dev -DAnypoint.uri=${DEVOPS_MULE_ANYPOINT_URI} -Dcloudhub.businessGroupId=${DEVOPS_CLOUDHUB_BUSINESSGROUPID} -Dcloudhub.connectedAppClientId=$DEVOPSUSERNAME -Dcloudhub.connectedAppClientSecret=$DEVOPSPASSWORD -Dcloudhub.connectedAppGrantType=${DEVOPS_CLOUDHUB_CONNECTEDAPPGRANTTYPE} -Dcloudhub.workerType=${DEVOPS_CLOUDHUB_WORKERTYPE} -Dcloudhub.workers=${DEVOPS_CLOUDHUB_WORKERS} -Dcloudhub.environment=${DEVOPS_CLOUDHUB_ENVIRONMENT} -Dregion=${DEVOPS_REGION}' 																
							}
							echo "Deployment completed"
					   } 
				   }
				}
			}				   
	    }
	  }
