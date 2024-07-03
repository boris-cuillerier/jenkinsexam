pipeline {
    agent any
    
    environment {
        DOCKER_ID = "bobdatascientest/jenkinsexam" // Replace with your Docker Hub ID
        DOCKER_COMPOSE_FILE = "docker-compose.yml"
        HELM_CHART_NAME = "examjenkins"
        HELM_CHART_VERSION = "0.1.0"
        KUBECONFIG_SECRET = credentials("config") // Jenkins credential ID for kubeconfig file
    }
    
    stages {
        stage('Build and Push Docker Images') {
            steps {
                script {
                    // Build images from docker-compose.yml
                    sh "docker-compose -f $DOCKER_COMPOSE_FILE build"
                    
                    // Tag and push images to Docker Hub
                    sh "docker-compose -f $DOCKER_COMPOSE_FILE push"
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

