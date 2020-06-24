// Basic Pipeline

pipeline {
    environment {
         REGISTRY_URL = "us.icr.io"
         MAJOR_PREFIX = "1.0.0"
         RELEASE_NAME = "guestbook"
         DEPLOYMENT_NS = "default"
         RESOURCE_GROUP = "default"
    }

    agent any

    stages {
         stage ('Initialize') {
            steps {
                sh '''#!/bin/bash
                  if [ -z "${API_KEY}"  ]; then
                    echo "FATAL ERROR: API_KEY undefined, exiting pipeline"
                    exit 22
                 else 
                    echo "API_KEY is set to (shhh, its a secret)"
                 fi
                 if [ -z "${CLUSTER_NAME}"  ]; then
                    echo "FATAL ERROR: CLUSTER_NAME undefined, exiting pipeline"
                    exit 22
                 else 
                    echo "CLUSTER_NAME set to ${CLUSTER_NAME}"
                 fi
                 if [ -z "${REGISTRY_NS}"  ]; then
                    echo "FATAL ERROR: REGISTRY_NS undefined, exiting pipeline"
                    exit 22
                 else 
                    echo "REGISTRY_NS set to ${REGISTRY_NS}"
                 fi
                 if [ -z "${REGION}"  ]; then
                    echo "FATAL ERROR: REGION undefined, exiting pipeline"
                    exit 22
                 else 
                    echo "REGION set to ${REGION}"
                 fi
                 if [ -z "${RESOURCE_GROUP}"  ]; then
                    echo "FATAL ERROR: RESOURCE_GROUP undefined, exiting pipeline"
                    exit 22
                 else 
                    echo "RESOURCE_GROUP set to ${RESOURCE_GROUP}"
                 fi
              
                 if [ -z "${DEPLOYMENT_NS}"  ]; then
                    echo "FATAL ERROR: DEPLOYMENT_NS undefined, exiting pipeline"
                    exit 22
                 else 
                    echo "DEPLOYMENT_NS set to ${DEPLOYMENT_NS}"
                 fi
                
                 if [ -z "${RELEASE_NAME}"  ]; then
                    echo "FATAL ERROR: RELEASE_NAME undefined, exiting pipeline"
                    exit 22
                 else 
                    echo "RELEASE_NAME set to ${RELEASE_NAME}"
                 fi
                '''
            }
         }
        
      stage('Checkout Code') {
         steps {
            checkout scm
         }
      }

       stage('Build Docker Image') {
            steps {
                script {
                   echo "docker build -t ${REGISTRY_URL}/${REGISTRY_NS}/guestbook:${MAJOR_PREFIX}.${BUILD_NUMBER} ./v1/guestbook"
                   sh 'docker build -t ${REGISTRY_URL}/${REGISTRY_NS}/guestbook:${MAJOR_PREFIX}.${BUILD_NUMBER} ./v1/guestbook'
                }

            }
       }

       stage('Push Docker Image to Registry') {
          steps {
                 sh """#!/bin/bash
                 docker login -u iamapikey -p ${API_KEY} ${env.REGISTRY_URL}
                 docker push ${env.REGISTRY_URL}/${env.REGISTRY_NS}/guestbook:${env.MAJOR_PREFIX}.${env.BUILD_NUMBER}
                 """
           }
       }

        stage('Deploy new Docker Image') {
            steps {
                echo 'Deploying....'
                    sh """#!/bin/bash
                    whoami
                    which ibmcloud
                    ibmcloud login --apikey ${API_KEY} -r ${env.REGION} -g ${env.RESOURCE_GROUP}
                    eval \$(ibmcloud ks cluster config --cluster ${env.CLUSTER_NAME})
                    DEPLOYMENT=`kubectl --namespace=${env.DEPLOYMENT_NS} get deployments -l app=guestbook --no-headers  -o name`
                    if [ -z "\$DEPLOYMENT"  ]; then
                       echo "Deployment not found, deploying for the first time"
                       kubectl apply -f ./v1/.
                    else
                       echo "Existing deployment found, updating to point to image just created "
                       kubectl --namespace=${env.DEPLOYMENT_NS} get \${DEPLOYMENT} --no-headers -o custom-columns=":metadata.name"

                       # Update Deployment
                       kubectl --namespace=${env.DEPLOYMENT_NS} set image \${DEPLOYMENT} ${env.RELEASE_NAME}=${env.REGISTRY_URL}/${env.REGISTRY_NS}/guestbook:${env.MAJOR_PREFIX}.${env.BUILD_NUMBER}
                       kubectl --namespace=${env.DEPLOYMENT_NS} rollout status \${DEPLOYMENT}
                    fi
                    """

            }
        }
    }
}