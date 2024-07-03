pipeline {
    agent any

    environment {
        DOCKER_ID = "bobdatascientest/jenkinsexam" // Replace with your Docker Hub ID
        DOCKER_COMPOSE_FILE = "docker-compose.yml"
        HELM_CHART_NAME = "examjenkins"
        HELM_CHART_VERSION = "0.1.0"
        KUBECONFIG_SECRET = credentials("config") // Jenkins credential ID for kubeconfig file
        DOCKER_TAG = ""
        GIT_CREDENTIALS = credentials('GIT_CREDENTIALS') // Jenkins credential ID for Git credentials
        DOCKER_HUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIALS') // Jenkins credential ID for Docker Hub credentials
    }

    stages {
        stage('Build and Push Docker Images') {
            steps {
                script {
                    // Use Docker Hub credentials to login
                    withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
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

