pipeline {
	environment {
		APP_NAME = "Elie"
		ID_DOCKER = "dockerd452"
		IMAGE_NAME = "alpinehelloworld"
		IMAGE_TAG = "latest"
		PORT_EXPOSED = "80"
		INTERNAL_PORT = "5000"
		EXTERNAL_PORT = "${PORT_EXPOSED}"
		STG_API_ENDPOINT = "192.168.56.9:1993"
       	STG_APP_ENDPOINT = "192.168.56.9:80"
       	CONTAINER_IMAGE = "${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}"
	}
	agent none
	stages {
		stage('BUILD image') {
			agent any
			steps {
				script {
					sh 'docker build -t $ID_DOCKER/$IMAGE_NAME:$IMAGE_TAG .'
				}
			}
		}
		stage('RUN container based on build image') {
			agent any
			steps {
				script {
					sh '''
						docker run --name $IMAGE_NAME -d -p $PORT_EXPOSED:$INTERNAL_PORT -e PORT=$INTERNAL_PORT $ID_DOCKER/$IMAGE_NAME:$IMAGE_TAG
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

		stage ('Login and Push Image on docker hub') {
          agent any
        	environment {
           		DOCKERHUB_CREDENTIALS = credentials('docker_dockerd452')
        	}            
          	steps {
            	script {
               		sh '''
                		echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    	docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
               		'''
             	}
          	}
      	}

      	stage('STAGING - Deploy app') {
       		when {
        		expression { GIT_BRANCH == 'origin/master' }
        	}
      		agent any

      		steps {
        		script {
            		sh '''
               			curl -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json' -d '{"your_name":"${APP_NAME}","container_image":"${CONTAINER_IMAGE}", "external_port":"${EXTERNAL_PORT}", "internal_port":"${INTERNAL_PORT}"}'
               		'''
        		}
        	}
     	} 
	}
}
