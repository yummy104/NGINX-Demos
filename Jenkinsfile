pipeline {
    environment {
        PROJECT = 'assuring-cockatoo-c4d6'
        APP_NAME = 'nginx'
        NAMESPACE = 'irc'
        KEN = 'KubernetesEngineBuilder'
        FE_SVC_NAME = "${APP_NAME}-service"
        CLUSTER = 'primary'
        CLUSTER_ZONE = 'us-west4'
        IMAGE_TAG = "us.gcr.io/cosmic-lion-8b1f/${APP_NAME}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
        JENKINS_CRED = "${PROJECT}"
    }

    agent {
        kubernetes {
            label 'nginx-app'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  containers:
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
        }
    }
    stages {
        stage('Build and push image with Container Builder') {
            steps {
                container('gcloud') {
                    sh "gcloud container images list"
                    sh "gcloud builds submit -t ${IMAGE_TAG} ."
                }
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                container('kubectl') {
                    sh("sed -i.bak 's#us.gcr.io/cosmic-lion-8b1f/nginx:1.0.0#${IMAGE_TAG}#' ./k8s/production/*.yaml")
                    step([
                        $class: env.KEB,
                        namespace: env.NAMESPACE,
                        projectId: env.PROJECT,
                        clusterName: env.CLUSTER,
                        zone: env.CLUSTER_ZONE,
                        manifestPattern: 'k8s/services',
                        credentialsId: env.JENKINS_CRED,
                        verifyDeployments: false
                    ])
                    step([
                        $class: env.KEB,
                        namespace: env.NAMESPACE,
                        projectId: env.PROJECT,
                        clusterName: env.CLUSTER,
                        zone: env.CLUSTER_ZONE,
                        manifestPattern: 'k8s/production',
                        credentialsId: env.JENKINS_CRED,
                        verifyDeployments: true
                    ])
                }
            }
        }
    }
}
