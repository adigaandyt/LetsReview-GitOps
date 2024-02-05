# Our_Library GitOps 
This repository contains the GitOps configuration for the [Our_Library WebApp](http://ourlib.ddns.net/), using ArgoCD to manage deployments to an EKS cluster. The configuration defines the desired state of our applications and infrastructure, ensuring that our deployment environment is declarative, version-controlled, and automated. ArgoCD is currently hosted [here](http://ourlibargo.ddns.net/) for demonstration purposes.

ArgoCD deploys the following using the **app of apps pattern**:
- Our_Library Application
  - The App's Helm Chart
  - MongoDB Helm Chart
  - AWS EBS CSI Driver
- Nginx Controller
- Cert Manager (to be added)
- Kube Prometheus Stack (to be added)
- Elasticsearch Fluentd Kibana Stack (to be added)

## Prerequisites
- An EKS Cluster running (could also use other Kubernetes clusters you just won't need the AWS EBS CSI Driver)
- ArgoCD installed on your cluster
- Credentials for the ArgoCD to access this repo

## Usage
Deploy ArgoCD on your cluster and apply infra-apps-app.yaml or use the ArgoCD UI and use the + New App option, copy paste the yaml file into the EDIT AS YAML section

## GitOps App of apps pattern
A simple diagram breakdown of the app of apps setup

![App of apps diagram](/diagrams/App%20of%20apps.png)

## GitOps Workflow

This repo is a part of the CI/CD deployment for Our_Library webapp, ArgoCD is in charge of the CD portion while Jenkins handles CI

![App of apps diagram](/diagrams/GitOps%20flow.png)
### Diagram breakdown
1) Devs push code to the application repo
2) App Repo sends a webhook to Jenkins announcing there's new code
3) Jenkins pulls the code from repo and builds an image using the Dockerfile that's on the repo
4) Jenkins runs health check and a test script to see that the app and all of it's API calls are working
5) If the image passes all the tests it gets an update tag based on the previous tag that's on the App Repo (1.0.6 -> 1.0.7)
6) Image gets pushed to ECR
7) Jenkins pulls the GitOps repo code, updates the image tag in values.yaml and pushes it back onto the GitOps Repo
8) GitOps repo sends a webhook to ArgoCD to announce there's a new version
9) ArgoCD sees that the manifest has changed so it updates it on the EKS Cluster
10) EKS Cluster pulls image from ECR and deploys it replacing the old deplyoment

Using a workflow like this means that the only thing developers have to do is push code onto the repo, Jenkins and ArgoCD will handle the testing and deployment automatically 
