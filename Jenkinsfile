pipeline {

    environment {
       imageName="drop1/web-login"
       DOCKERHUB_CREDENTIALS= "mydkr-pat"
       PROJECT_ID="compelling-cat-431716-m7"
       CLUSTER_NAME="autopilot-cluster-1"
       location="us-central1"
       GCP_CREDENTIALS_ID="stb-sa"
       HELM_RELEASE = 'web-login'
       HELM_CHART_PATH = './charts/web-login'
       HELM_VALUES_PATH = './h-override/override-values.yaml'
       NAMESPACE = 'default'

    }
  agent any
  stages {

      stage('image bake ') {
            steps {
              script {
                 docker.build("${imageName}:${env.BUILD_NUMBER}")
              }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: DOCKERHUB_CREDENTIALS, variable: 'DOCKER_TOKEN')]) {
                        sh "echo ${DOCKER_TOKEN} | docker login -u hareeshrao4839 --password-stdin"
                        docker.image("${imageName}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
      stage('helm-validate ') {
            steps {
              script {
                sh """
                #/!bin/bash
                helm template web-login-v1 charts/web-login \
                -f h-override/override-values.yaml \
                --set image.tag="${env.BUILD_NUMBER}"
                """
              }
            }
        }
      stage('setup gcloud ') {
            steps {
              script {
                sh """
                #/!bin/bash
                gcloud container clusters get-credentials ${env.CLUSTER_NAME} --region ${env.LOCATION} --project ${env.PROJECT_ID}
                kubectl get ns
                """
              }
            }
        }

        stage('Deploy to Kubernetes with Helm') {
            steps {
                script {
                    sh """
                    helm upgrade --install ${env.HELM_RELEASE} ${env.HELM_CHART_PATH} \
                    -f ${env.HELM_VALUES_PATH} --namespace ${env.NAMESPACE} \
                    --force --create-namespace '--set=image.tag=${env.BUILD_NUMBER}'
                    """
                    echo "Helm chart deployment finished..."
                    sh "helm status ${env.HELM_RELEASE} --namespace ${env.NAMESPACE} "
                }
            }
        }
        }
    //  post {
    //     always {
    //         cleanWs()
    //     }
    //     }
}

