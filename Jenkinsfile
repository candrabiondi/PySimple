pipeline {
	agent {
		node {label 'python'}
	}
	environment {
		APP_NAME = 'PySimple'
		GIT_REPO = 'https://github.com/candrabiondi/PySimple.git'
		GIT_BRANCH = 'master'
		DEV = 'dev'
		STAGE = 'stage'
		TEMPLATE_NAME = 'python'
		ARTIFACT_FOLDER = "target"
		PORT = 8080
	}
	stages {
		stage('Get Latest Code'){
			steps {
				git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
			}
		}
		stage("Install Dependencies"){
			steps {
				sh """
				pip install virtualenv
				pip install --upgrade virtualenv==16.7.9
				virtualenv --no-site-packages .
				source bin/activate
				pip install -r requirements.txt
				deactivate 
				"""
			}
		}
//		stage('Run Tests') {
//			steps {
//				sh '''
//				source bin/activate
//				nosetests app --with-xunit
//				deactivate
//				'''
//				junit "nosetests.xml"
//			}
//		}
		stage('Archive App'){
			steps{
				script{
					def safeBuildName  = "${APP_NAME}_${BUILD_NUMBER}",
                        		artifactFolder = "${ARTIFACT_FOLDER}",
                        		fullFileName   = "${safeBuildName}.tar.gz",
                        		applicationZip = "${artifactFolder}/${fullFileName}"
                        		applicationDir = ["Dockerfile",
                                            		 ].join(" ");
                    			def needTargetPath = !fileExists("${artifactFolder}")
                    			if (needTargetPath) {
                        		sh "mkdir ${artifactFolder}"
                    			}
                    			sh "tar -czvf ${applicationZip} ${applicationDir}"
                    			archiveArtifacts artifacts: "${applicationZip}", excludes: null, onlyIfSuccessful: true
				}
			}
		}
		stage('Create image Builder'){
			when {
				expression {
					openshift.withCluster() {
						openshift.withProject(DEV){
							return !openshift.selector("bc", "${TEMPLATE_NAME}").exists();
						}
					}
				}
			}
			steps {
				script {
					openshift.withCluster() {
						openshift.withProject(env.DEV) {
							openshift.newBuild("--name=${TEMPLATE_NAME}", "--docker-image=docker.io/python:3.10.1-alpine3.15", "--binary=true")
						}
					}
				}
			}
		}
		stage('Build Image') {
            		steps {
                		script {
                    			openshift.withCluster() {
                        			openshift.withProject(env.DEV) {
                           	 			openshift.selector("bc", "$TEMPLATE_NAME").startBuild("--from-archive=${ARTIFACT_FOLDER}/${APP_NAME}_${BUILD_NUMBER}.tar.gz", "--wait=true")
                        			}
                    			}
                		}
            		}
       	 	}
		stage('Deploy to DEV') {
            		when {
                		expression {
                    			openshift.withCluster() {
                        			openshift.withProject(env.DEV_PROJECT) {
                            				return !openshift.selector('dc', "${TEMPLATE_NAME}").exists()
                        			}
                    			}
                		}
            		}
            		steps {
                		script {
                    			openshift.withCluster() {
                        			openshift.withProject(env.DEV_PROJECT) {
                            				def app = openshift.newApp("${TEMPLATE_NAME}:latest")
                            				app.narrow("svc").expose("--port=${PORT}");
                            				def dc = openshift.selector("dc", "${TEMPLATE_NAME}")
                            				while (dc.object().spec.replicas != dc.object().status.availableReplicas) {
                               	 				sleep 10
                            				}
                        			}
                    			}
                		}
            		}
        	}
	}
}

