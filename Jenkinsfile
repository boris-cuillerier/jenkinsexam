pipeline {
    agent any

    environment {
        DOCKER_ID = "bobdatascientest/jenkinsexam" // Replace with your Docker Hub ID
        DOCKER_PASS
	DOCKER_COMPOSE_FILE = "docker-compose.yml"
        HELM_CHART_NAME = "examjenkins"
        HELM_CHART_VERSION = "0.1.0"
        KUBECONFIG_SECRET = credentials("config") // Jenkins credential ID for kubeconfig file
        DOCKER_TAG = ""
    }

    stages {
        stage('Build and Push Docker Images') {
	    environment {
                DOCKER_PASS = credentials('DOCKER_HUB_PASS')
            }
            steps {
                script {
                    sh """
		    docker login -u $DOCKER_ID -p $DOCKER_PASS
		    docker push $DOCKER_ID/$HELM_CHART_NAME:$DOCKER_TAG
	            """

		    // Build and push images using docker-compose
                    sh "docker-compose -f $DOCKER_COMPOSE_FILE build"
                    sh "docker-compose -f $DOCKER_COMPOSE_FILE push"
                    
                    // Tag images with the current build number
                    DOCKER_TAG = "v.${BUILD_ID}.0"
                    sh "docker tag examjenkins_cast_service:latest $DOCKER_ID/$HELM_CHART_NAME:$DOCKER_TAG"
                    sh "docker push $DOCKER_ID/$HELM_CHART_NAME:$DOCKER_TAG"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Loop through namespaces and deploy with Helm
                    def namespaces = ["dev", "qa", "staging", "prod"]

                    for (def namespace in namespaces) {
                        sh """
                        cp examjenkins/values.yaml values-${namespace}.yaml
                        sed -i "s+tag: .*+tag: ${DOCKER_TAG}+g" values-${namespace}.yaml
                        helm upgrade --install $HELM_CHART_NAME ./examjenkins-${HELM_CHART_VERSION}.tgz --values=values-${namespace}.yaml --namespace $namespace
                        """
                    }
                }
            }
        }
    }
}

