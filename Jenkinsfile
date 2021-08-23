pipeline {
	environment {
		PROD1_SERVER_IP = "192.168.1.106"
		PROD2_SERVER_IP = "192.168.1.108"
		DEV1_SERVER_IP = "192.168.1.105"
		DEV2_SERVER_IP = "192.168.1.103"
		GIT_COMMIT_SHORT = sh(
                script: "printf \$(git rev-parse --short HEAD)",
                returnStdout: true
        )
        def VERSION = getArtifactVersion(GIT_COMMIT_SHORT)
        def ARTIFACT = "config-service-${VERSION}.jar"
	}
    //agent { label 'slave01' }
	agent any
    stages{
		stage ('Initialize') {
            steps {
                sh '''
                    echo "PROD1_SERVER_IP = ${PROD1_SERVER_IP}"
                    echo "PROD2_SERVER_IP = ${PROD2_SERVER_IP}"
                    echo "DEV1_SERVER_IP = ${DEV1_SERVER_IP}"
                    echo "DEV2_SERVER_IP = ${DEV2_SERVER_IP}"
                    echo "GIT_COMMIT_SHORT = ${GIT_COMMIT_SHORT}"
					echo "VERSION = ${VERSION}"
					echo "ARTIFACT = ${ARTIFACT}"
                '''
            }
        }
        stage('Clone repository') {
			steps {
				script {
					/* Let's make sure we have the repository cloned to our workspace */
					checkout scm
				}
			}
		}
		stage('Build for development') {
			when {
                branch 'develop'
            }
			steps {
				script {
					withMaven(mavenSettingsConfig: 'MyMavenSettings') {
						sh 'env' 
		      			  sh '''
					     	mvn -B org.codehaus.mojo:versions-maven-plugin:2.8.1:set -DprocessAllModules -DnewVersion=${VERSION}
					        mvn -U clean deploy
					     '''
					}
				}
			}
		}
		
	   stage('Stopping dev config service dev') {
	   	when {
                branch 'develop'
            }
	   		steps {
		      	script {
		      		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
		      			sh "ssh jenkins@$DEV1_SERVER_IP sudo systemctl stop dvdtheque-server-config.service"
			      		sh "ssh jenkins@$DEV2_SERVER_IP sudo systemctl stop dvdtheque-server-config.service"
		      		}
		      	}
		   }
	   }
	   stage('Stopping production config service dev') {
	   	when {
                branch 'master'
            }
	   		steps {
		      	script {
		      		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
		      			sh "ssh jenkins@$PROD1_SERVER_IP sudo systemctl stop dvdtheque-server-config.service"
			      		sh "ssh jenkins@$PROD2_SERVER_IP sudo systemctl stop dvdtheque-server-config.service"
		      		}
		      	}
		   }
	   }
	   stage('Copying dev config service jar') {
	   		when {
                branch 'develop'
            }
	   		steps {
		      	script {
		      		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
				        sh "scp target/$ARTIFACT jenkins@$DEV1_SERVER_IP:/opt/dvdtheque_server_config_service/dvdtheque-server-config.service.jar"
				        sh "scp target/$ARTIFACT jenkins@$DEV2_SERVER_IP:/opt/dvdtheque_server_config_service/dvdtheque-server-config.service.jar"
		      		}
		      	}
		    }
	   }
	   stage('Copying production config service jar') {
	   		when {
                branch 'master'
            }
	   		steps {
		      	script {
		      		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
				        sh "scp discovery-service/target/$ARTIFACT jenkins@$PROD1_SERVER_IP:/opt/dvdtheque_server_config_service/config-service.jar"
				        sh "scp discovery-service/target/$ARTIFACT jenkins@$PROD2_SERVER_IP:/opt/dvdtheque_server_config_service/config-service.jar"
		      		}
		      	}
		    }
	   }
	   stage('Sarting dev config service') {
	   		when {
                branch 'develop'
            }
	   		steps {
		      	script {
		      		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
		      			sh "ssh jenkins@$DEV1_SERVER_IP sudo systemctl start dvdtheque-server-config.service"
				        sh "ssh jenkins@$DEV2_SERVER_IP sudo systemctl start dvdtheque-server-config.service"
		      		}
		      	}
		    }
	   }
	   stage('Sarting production config service') {
	   		when {
                branch 'master'
            }
	   		steps {
		      	script {
		      		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
		      			sh "ssh jenkins@$PROD1_SERVER_IP sudo systemctl start dvdtheque-server-config.service"
				        sh "ssh jenkins@$PROD2_SERVER_IP sudo systemctl start dvdtheque-server-config.service"
		      		}
		      	}
		    }
	   }
	   stage('Check dev status config service') {
	   		when {
                branch 'develop'
            }
	   		steps {
		      	script {
		      		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
		      			sh "ssh jenkins@$DEV1_SERVER_IP sudo systemctl status dvdtheque-server-config.service"
		      			sh "ssh jenkins@$DEV2_SERVER_IP sudo systemctl status dvdtheque-server-config.service"
		      		}
		      	}
		    }
	   }
	   stage('Check productrion status config service') {
	   		when {
                branch 'master'
            }
	   		steps {
		      	script {
		      		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
		      			sh "ssh jenkins@$PROD1_SERVER_IP sudo systemctl status dvdtheque-server-config.service"
		      			sh "ssh jenkins@$PROD2_SERVER_IP sudo systemctl status dvdtheque-server-config.service"
		      		}
		      	}
		    }
	   }
    }
}

private String getArtifactVersion(String gitRevision){
	def gitBranchName
	gitBranchName = env.GIT_BRANCH
	if(gitBranchName == "develop"){
		return "${gitRevision}-SNAPSHOT"
	}
	if(gitBranchName == "master"){
		gitTagName = sh script: "git describe --tags --all ${gitRevision}", returnStdout: true
		return "${gitRevision}"
	}
	return ""
}