/*
    An example simple CI/CD of a node web app "from git to Kubernetes"ץ
    This pipeline assumes you Jenkins is setup with docker, kubectl, and helm for executing all steps of the CI/CD.
 */


pipeline {
    options {
        // Build auto timeout
        timeout(time: 60, unit: 'MINUTES')
    }

    // Some global default variables.
    // You can convert some to parameters to give better control of the flow.
    environment {
        nodePath = 'node'
        repository = 'demo'
        registry = 'docker-artifactory-webinar.jfrogdev.co'
        creds = 'dockerregistrywebinar'
        artCreds = credentials('dockerregistrywebinar')

        helmRepo = 'https://artifactory-webinar.jfrogdev.co/artifactory/helm'
        helmRelease = 'webinar-example'

        dockerTag = '0.0'
        helmChartVersion = '0.0.1'
    }

    // In this example, all is built and run from the master
    agent { node { label 'master' } }

    // Pipeline stages
    stages {

        /*
        Step 1

        Build the Docker image of the node web app and run a local test (curl)
        */
        stage('Docker build') {
            steps {
                script {
                    echo "========== Git clone =========="
                    git branch: 'master',
                            url: 'git@github.com:eldada/node-git-to-k8s.git'

                    echo "========== Docker build =========="
                    def fullImage = "${registry}/${repository}:${dockerTag}.${env.BUILD_NUMBER}"

                    def dockerImage = docker.build(fullImage, "--build-arg NPM_AUTH=${NPM_AUTH} .")

                    echo "========== Test Docker image =========="
                    // Test Docker image
                    docker.image(fullImage).withRun('--name node-test -p 8888:8080') { c ->
                        sh 'sleep 3'
                        sh 'curl -s http://localhost:8888'
                    }

                    echo "========== Docker push =========="
                    echo "Pushing to ${registry}"
                    docker.withRegistry( "http://${registry}", creds ) {
                        dockerImage.push()
                    }
                }
            }
        }

        /*
        Step 2

        Package helm chart and publish to Artifactory
         */
        stage('Helm package') {
            steps {
                script {
                    echo "========== Helm package =========="
                    sh "helm package demo"

                    echo "========== Publish helm chart =========="
                    sh "curl -u${artCreds} -T ./demo-${helmChartVersion}.tgz '${helmRepo}/demo-${helmChartVersion}.tgz'"
                }
            }
        }

        /*
         Step 3

         Deploy to Kubernetes and run test

         - Assumes kubectl and helm client already configured
         - Assumes a private 'helm' repository is already configured in Artifactory
          */
        stage('Deploy') {
            steps {
                script {
                    echo "========== Helm repo add =========="
                    sh "helm repo add --username ${artCreds_USR} --password ${artCreds_PSW} webinar ${helmRepo}"

                    echo "========== Deploy =========="
                    sh "helm upgrade --install ${helmRelease} --set image.tag=${dockerTag}.${env.BUILD_NUMBER} webinar/demo"

                    echo "========== Status =========="
                    sh 'sleep 10'
                    sh "helm status ${helmRelease}"
                }
            }
        }
    }
}

