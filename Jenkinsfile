pipeline {
     agent {
        node {
            label 'Agent01'
        }
    }

	  tools {
          jdk 'jdk17.0'
    }
    environment {
	     APP_NAME = "pdp-pep"
        DOCKER_IMAGE = 'server' 
	ARTIFACTORY_SERVER = "harbor.tango.rid-intrasoft.eu"
      ARTIFACTORY_DOCKER_REGISTRY = "harbor.tango.rid-intrasoft.eu/pdp-pep/"
      BRANCH_NAME = "main"
      DOCKER_IMAGE_TAG = "$APP_NAME:R${env.BUILD_ID}"
    }
   stages {
        stage('Compile') {
            steps {
                dir('demo') {
		    sh 'java -version'
		    sh 'echo "JAVA_HOME=$JAVA_HOME"'
		    sh './gradlew build'
                }
            }
        }
	        stage('Build image') { // build and tag docker image
            steps {
		       dir('demo') {
                echo 'Starting to build docker image'
                script {
                    def dockerImage = docker.build(ARTIFACTORY_DOCKER_REGISTRY + DOCKER_IMAGE_TAG) 
                }
            }
	    }
        }

	stage("Push_Image"){
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'harbor-jenkins-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]){
                    echo "***** Push Docker Image *****"
                    sh 'docker login ${ARTIFACTORY_SERVER} -u ${USERNAME} -p ${PASSWORD}'
                    sh 'docker image push ${ARTIFACTORY_DOCKER_REGISTRY}${DOCKER_IMAGE_TAG}'
		    sh 'docker tag ${ARTIFACTORY_DOCKER_REGISTRY}${DOCKER_IMAGE_TAG} ${ARTIFACTORY_DOCKER_REGISTRY}${APP_NAME}:latest_dev'
		    sh 'docker image push ${ARTIFACTORY_DOCKER_REGISTRY}${APP_NAME}:latest_dev'
                }
            }
        }
	         stage('Docker Remove Image locally') {
        steps {
                sh 'docker rmi "$ARTIFACTORY_DOCKER_REGISTRY$DOCKER_IMAGE_TAG"'
		sh 'docker rmi "$ARTIFACTORY_DOCKER_REGISTRY$APP_NAME:latest_dev"'
            }
        }
	 stage('Build') {
            steps {
                sh 'docker build -t myapp_img .'
            }
        }
	   stage("Run server"){
		    steps {
			     dir('demo') {
				    sh '''
                    docker run -d --name myapp \
                      --network host \
                      -v "$(pwd)/build/libs:/app/build/libs" \
                      -v "$(pwd)/crypto:/app/crypto" \
                      -v "$(pwd)/ec-cakey.jks:/app/ec-cakey.jks" \
                      -v "$(pwd)/ec-cacert.pem:/app/ec-cacert.pem" \
                      -v "$(pwd)/resources:/app/resources" \
                      -v "$(pwd)/temperatura:/app/temperatura" \
                      --env-file .env \
                      -e PDP_PORT=${PDP_PORT} \
                      -e PDP_CONFIG=${PDP_CONFIG} \
                      -e PDP_KS=${PDP_KS} \
                      -e PDP_PW=${PDP_PW} \
                      -e PDP_ALIAS=${PDP_ALIAS} \
                      -e DLT_IP=${DLT_IP} \
                      -e DLT_PORT=${DLT_PORT} \
                      -e IDAGENT_KS=${IDAGENT_KS} \
                      -e IDAGENT_PW=${IDAGENT_PW} \
                      -e IDAGENT_ALIAS=${IDAGENT_ALIAS} \
                      -e IDAGENT_CERT=${IDAGENT_CERT} \
                      -e IDAGENT_PORT=${IDAGENT_PORT} \
                      -e IDAGENT_IP=${IDAGENT_IP} \
                      -e RESOURCES=${RESOURCES} \
                      myapp_img
                '''
				    }
		    }
	   }
   }
}
