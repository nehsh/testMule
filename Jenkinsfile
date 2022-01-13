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
		    stage('code quality check'){
			    steps {
				    script {
					    configFileProvider([configFile(fileId: '5bddc05b-6648-42da-8916-73b4a94c5abb', variable: 'Maven_Settings1')]) {
						  echo "sonar check"
						  sh "mvn -s $Maven_Settings1 -Psonar:sonar"
					    }}}}					    
		    
		    stage('deploy to exchange'){
		     steps {
			script{
				//pom = readMavenPom(file: 'pom.xml')		
			        //POM_ARTIFACTID = pom.getArtifactId().toString()
				configFileProvider([configFile(fileId: '5bddc05b-6648-42da-8916-73b4a94c5abb', variable: 'Maven_Settings1')]) {
				withCredentials([usernamePassword(credentialsId: 'anypoint-platform', usernameVariable: 'DEVOPSUSERNAME', passwordVariable: 'DEVOPSPASSWORD')]) {
				sh "echo ${DEVOPSUSERNAME}"
				sh "mvn -s $Maven_Settings1 clean package deploy"
				echo "exchange completed"
			      }}}}}
		    
		    
						
		    stage('deploy to rtf'){
			    steps {
			script{
				//pom = readMavenPom(file: 'pom.xml')		
			        //POM_ARTIFACTID = pom.getArtifactId().toString()
				configFileProvider([configFile(fileId: '5bddc05b-6648-42da-8916-73b4a94c5abb', variable: 'Maven_Settings1')]) {
				withCredentials([usernamePassword(credentialsId: 'anypoint-platform', usernameVariable: 'DEVOPSUSERNAME', passwordVariable: 'DEVOPSPASSWORD')]) {
				sh "echo ${DEVOPSUSERNAME}"
				sh "mvn -s $Maven_Settings1 deploy -DmuleDeploy -DskipMunitTests -Dcloudhub.muleVersion=${DEVOPS_CLOUDHUB_MULEVERSION} -Dcloudhub.applicationName=test-mule-dev-rtf-api -DAnypoint.uri=${DEVOPS_MULE_ANYPOINT_URI} -Dcloudhub.businessGroupId=${DEVOPS_CLOUDHUB_BUSINESSGROUPID} -Dcloudhub.connectedAppClientId=$DEVOPSUSERNAME -Dcloudhub.connectedAppClientSecret=$DEVOPSPASSWORD -Dcloudhub.connectedAppGrantType=${DEVOPS_CLOUDHUB_CONNECTEDAPPGRANTTYPE}"
				}}}}}
				}
	
	  }
