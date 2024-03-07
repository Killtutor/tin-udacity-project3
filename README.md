# DEVOPS udacity project by tincito

## What is this project

This project consist of a DB in postgres SQL deployed in a Kubernetes cluster in AWS EKS with and a python Service that allows users to get analytics from the data stored in the DB.
The database is deployed with Helm chart and the analytics service is deployed with a docker image as a service in our kubernetes cluster

## Technologies Used

- Github as our code versioning control and trigger for our CICD pipeline
- AWS as our cloud provider
- AWS codeBuild as our docker builder
- AWS Elastic Container Registry as our docker image repository
- AWS Elastic Kubernetes Service as our kubernete service

### How everything get to work?

The cluster should always be running, with the Database service up and running with the data you need to analyse you can connect to it from outside the cluster with a AWS IAM user in order to update, remove or add new data to the DB. Also you can deploy changes to the analytics service pushing code to the main branch of the server in which case will trigger a codebuild job to update the latest image for the service, if you are completetly sure that the image is working and you want to deploy it to production you just need to redeploy the service (analytics.svc.yml) to the cluster with the following command: kubectl rollout restart deployment/analytics-svc

### Debug

If you want to debug this project you can use kubectl logs deployment/analytics-svc or you can go to the AWS cloudwatch and seek the logs to debug the cluster apps
