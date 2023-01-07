pipeline {
	environment {
		IMAGE_NAME = "alipnehelloworld"
		IMAGE_TAG = "latest"
	}
	agent none
	stages {
		stage('BUILD image') {
			agent any
			steps {
				script {
					sh 'docker build -t dockerd452/$IMAGE_NAME:$IMAGE_TAG .'
				}
			}
		}
		stage('RUN container based on build image') {
			agent any
			steps {
				script {
					sh '''
						docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 dockerd452/$IMAGE_NAME:$IMAGE_TAG
						sleep 5
					'''
				}
			}
		}
		stage('TEST image') {
			agent any
			steps {
				script {
					sh '''
						curl http://172.17.0.1 | grep -q "Hello world!"
					'''
				}
			}
		}
		stage('Clean Container') {
			agent any
			steps {
				script {
					sh '''
						docker stop ${IMAGE_NAME}

						docker rm ${IMAGE_NAME}
					'''
				}
			}
		}
	}
}
