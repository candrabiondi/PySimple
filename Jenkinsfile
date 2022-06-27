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
				virtualenv --no-site-packages .
				source bin/activate
				pip install -r requirement.txt
				deactivate 
				"""
			}
		}
		stage('Run Tests') {
			steps {
				sh '''
				source bin/activate
				nosetest app --with-xunit
				deactivate
				'''
				junit "nosetests.xml"
			}
		}
		stage('Create image Builder'){
			when {
				exression {
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
							openshift.newBuild("--name=${TEMPLATE_NAME}", "--docker-image=docker.io/python:3.10.1-alpine3.15", "--binary=true"
						}
					}
				}
			}
		}
	}
}

