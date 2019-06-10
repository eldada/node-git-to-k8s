# CI/CD of NodeJS web app to Kubernetes using Jenkins, Docker and Helm
This repository has a very basic example for packing a [NodeJS](https://nodejs.org) web application in [Docker](https://www.docker.com/) and deploying it to [Kubernetes](https://kubernetes.io/) using [Helm](https://helm.sh/).
All this is done with [Jenkins](https://jenkins.io/).

## Details
This pipeline is executed with Jenkins and assumes it's already setup with 
- Docker
- Kubectl
- Helm

These tools are required and are not in the scope of this example.

## Pipeline
The [Jenkinsfile](Jenkinsfile) has a very basic pipeline for
1. Step 1 (Docker build)
    - Build node web app Docker image
    - Run simple test (`curl`)
    - Docker push to Artifactory
    
2. Step 2 (Helm package)
    - Package Helm chart
    - Upload Helm chart to Artifactory

3. Step 3 (Deploy)
    - Helm install demo app
    - Show deployment status
