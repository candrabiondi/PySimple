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
		stage('Store Artifact'){
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
						openshift.withProject(DEV_PROJECT){
							return !openshift.selector("bc", "${TEMPLATE_NAME}").exist();
						}
					}
				}
			}
			steps {
				script {
					openshift.withCluster() {
						openshift.withProject(DEV_PROJECT) {
							openshift.newBuild("--name=${TEMPLATE_NAME}", "--docker-image=docker.io/python:3.10.1-alpine3.15", "--binary=true")
						}
					}
				}
			}
		}
	}
}

