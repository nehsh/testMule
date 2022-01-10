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
          POM_VERSION = getPomVersion()
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
		    stage('Download from Nexus'){
			    steps{
				    script {
              pom = readMavenPom file: "pom.xml";
                        // Find built artifact under target folder
                       filesByGlob = findFiles(glob: "target/*${pom.packaging}*");
                        // Print some info from the artifact found
                        echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                        // Extract the path from the File found
                       artifactPath = filesByGlob[0].path;
                        // Assign to a boolean response verifying If the artifact name exists
                        artifactExists = fileExists artifactPath;

             if(artifactExists) {
                            echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
					    configFileProvider([configFile(fileId: 'b16cd1b3-6027-4042-af76-104f3e1f418e', variable: 'Maven_Settings')]){
						    echo "downloading the artifact from Nexus"
						    sh "cd $WORKSPACE"
						    sh "ls -lrta"
						    withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
						    sh "curl -u $USERNAME:$PASSWORD http://35.202.86.163:32001/repository/beam_mulesoft_devops/com/beam/suntory/integration/testMule/${pom.version}/${pom.artifactId}-${pom.version}-${pom.packaging}.jar --output Mulesoft.jar"
						    sh "ls -lrta"
					            sh "pwd"}}}
               else {
                            error "*** File: ${artifactPath}, could not be found";
                        }
				    
			    }    
			    }}
			stage('Build and Deploy') {
				steps {
					script {
						configFileProvider([configFile(fileId: 'b16cd1b3-6027-4042-af76-104f3e1f418e', variable: 'Maven_Settings')]) {
							echo "Deploying to Mule"
							echo 'Building project'
							withCredentials([usernamePassword(credentialsId: 'anypoint-platform', usernameVariable: 'DEVOPSUSERNAME', passwordVariable: 'DEVOPSPASSWORD')]) {
								sh "echo ${DEVOPSUSERNAME}"
								sh "cd $WORKSPACE"
								sh "pwd"
								sh 'mvn -s $Maven_Settings mule:deploy -Dmule.artifact=${WORKSPACE}/Mulesoft.jar -DskipMunitTests -Dcloudhub.muleVersion=${DEVOPS_CLOUDHUB_MULEVERSION} -Dcloudhub.applicationName=testMule-dev -DAnypoint.uri=${DEVOPS_MULE_ANYPOINT_URI} -Dcloudhub.businessGroupId=${DEVOPS_CLOUDHUB_BUSINESSGROUPID} -Dcloudhub.connectedAppClientId=$DEVOPSUSERNAME -Dcloudhub.connectedAppClientSecret=$DEVOPSPASSWORD -Dcloudhub.connectedAppGrantType=${DEVOPS_CLOUDHUB_CONNECTEDAPPGRANTTYPE} -Dcloudhub.workerType=${DEVOPS_CLOUDHUB_WORKERTYPE} -Dcloudhub.workers=${DEVOPS_CLOUDHUB_WORKERS} -Dcloudhub.environment=${DEVOPS_CLOUDHUB_ENVIRONMENT} -Dregion=${DEVOPS_REGION}' 																
							}
							echo "Deployment completed"
					   } 
				   }
				}
			}				   
	    }
	def getPomVersion(){
                     script{
                        pom = readMavenPom file: "pom.xml";
                        findFiles(glob: "target/*${pom.packaging}*");
                        pomversionName = pom.version;
                        return pomversionName;
                        }}
	  }
