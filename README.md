# LetsReview GitOps
This repository contains the GitOps configuration for [LetsReview](https://github.com/adigaandyt/LetsReview), using ArgoCD to manage deployments to an EKS cluster. The configuration defines the desired state of our applications and infrastructure, ensuring that our deployment environment is declarative, version-controlled, and automated.

It also containers the workflow of my entire DevOps Portfolio Project

ArgoCD deploys the following using the **app of apps pattern**:
- Our_Library Application
  - The App's Helm Chart
  - MongoDB Helm Chart
  - AWS EBS CSI Driver
- Nginx Controller
- Cert Manager 
- Kube Prometheus Stack 
- Elasticsearch Fluentd Kibana Stack

## Prerequisites
- An EKS Cluster running (could also use other Kubernetes clusters you just won't need the AWS EBS CSI Driver)
- ArgoCD installed on your cluster
- Credentials for the ArgoCD to access this repo

## Usage
Deploy ArgoCD on your cluster and apply infra-apps-app.yaml or use the ArgoCD UI and use the + New App option, copy paste the yaml file into the EDIT AS YAML section.
In this project ArgoCD get's installed using Terraform and a bootstrap application add the repo to ArgoCD automatically on startup
[IaC Repo](https://github.com/adigaandyt/ourlibrary-infra) 

## GitOps Workflow

This repo is a part of the CI/CD deployment for [LetsReview](https://github.com/adigaandyt/LetsReview), ArgoCD is in charge of the CD portion while Jenkins handles CI

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


# DevOps Portfolio Project
## Overview
This is here to summraize in phases the entire project in one place to demonstrate the technologies I used to a implement CI/CD pipeline

### Technology stack
- **Ticketing** - Trello to track tasks and missions for the day/week
- **Development** - Flask with REST API
- **Containerization** - Docker
- **Docker image registry**- ECR
- **Source Code Management** - GitHub
- **DataBase** - MongoDB
- **Scripting** - Bash
- **Infrastructure as Code** - Terraform
- **Cloud** - AWS
- **Kubernetes** - EKS
- **Kubernetes** packaging - Helm
- **Ingress** - Nginx ingress controller
- **CI/CD** - Jenkins
- **GitOps** - ArgoCD
- **Logging** and Monitoring - EFK stack, Prometheus, and Grafana

  
## Phase 0: Trello and diagrams
Before beginning to code or setting up anything, I first created a trello board to track my overall goals and daily goals and added bugs as they happen to keep track of my progress and to know what my next goal is.
I then drew a rough outline of the whole project and made a diagram encapsulating just to know roughly where I'm headed, the diagram changed and expanded as the project advanced.

## Phase 1: Repository Setup 
I set up 3 repositories for this project,
1) [LetsReview-App](https://github.com/adigaandyt/LetsReview-App) The application repo which contains
    - The flask app we're deploying
    - Dockerfile to create an image of the app
    - Docker-compose file to create a local environment for tests, with a diagram
    - Jenkinsfile for the CI pipeline
    - Test script for an E2E test for all the API calls


2) [LetsReview-Infra](https://github.com/adigaandyt/LetsReview-Infra) Infarstructure as code repo which containts
    -  Terraform code for the AWS Infarstratcture that will host the application
    -  Terraform modules for Network, EKS, Nodes and ArgoCD
    -  ArgoCD Values file
  
3) [LetsReview-GitOps](https://github.com/adigaandyt/LetsReview-GitOps) - GitOps repository used to manage the state of the cluster using this repository as the single source of truth and ArgoCD, utilizing an app of apps deployment pattern. It contains:
   - A `parent-app.yaml` file which acts as the parent application.
   - Helm charts for:
     - The application - The image is created and hosted on ECR, with the pipeline from [LetsReview-App](https://github.com/adigaandyt/LetsReview-App).
     - MongoDB - For stateful set pods as the database.
     - Cert-manager - For TLS management.
     - EFK Stack - For logging.
     - Kibana & Prometheus - For monitoring.
     - Ingress NGINX - To provide an ingress controller.

## Phase 3: Development
I wrote a simple single file flask application with a few REST API endpoints that does CRUD operations with a MongoDB database, and added a single HTML page to have NGINX serve static content, this wasn't the focus of the project so it was make fast and simple with GET/POST/PUT/DELETE requests and a /metrics endpoint for prometheus with log outputs for EFK

## Phase 4: Containerization
I wrote a dockerfile to containerize the application and to host it's image on ECR, and a docker-compose file for local testing, but because this is such a simple application I didn't use a multi-stage dockerfile, however, I did include a dockerfile for a different application that does have a multi-stage build just to demonstrate it

## Phase 5: CI/CD Pipeline
Wrote a Jenkinsfile to have a CI/CD Pipeline which has the following steps:
  -  Checkout application repo - Pull the application code
  -  Build - Run docker build 
  -  Unit Test - run the docker-compose to have a 3 tier application setup and run a health check
  -  E2E Test - using the compose from before run a test for every API request
  -  Hanlde version - check the latest tag and incriminite the patch by 1
  -  Tag Image and Github commit (application repo) - If the tests pass then add a version tag to the commit we pulled
  -  Push tagged image to ECR - Tag the image with the new patch and push it to ECR
  -  Push new value to GitOps repo - Push the tag to the values file on the GitOps repo to trigger ArgoCD to update our cluster's state
The Jenkins file uses a .env.groovy file to pass the nessacry values such as the image and repos needed so you can use it in a different project and functions to handle versioning and tags

## Phase 6: Helm and GitOps
I've provisioned a Kubernetes cluster using Terraform with AWS EKS for my application, I created the infrastructure using modules I wrote:
  -  **Network Module** - A module for provisiong network resources needed for the app which includes:
     - *VPC* - The virtual network that hosts everything, set cidr block based on user input
     - *Public Subnets* - Creates public subnets based on a Count input 
     - *Route Table* and Internet Gateway - for handling traffic to and inside the subnets
  -  **EKS Module** - A module that creates an EKS Cluster and an IAM Role for the cluster to assume, the role has the (AmazonEKSClusterPolicy)[https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html] attached (Amazon EKS use this role to manage nodes and services needed to run the cluster)
  -  **Nodes Module** - A module that creates a nodegroup (size based on input) and creates an IAM Role with a few polocies attached
     - [AmazonEKSWorkerNodePolicy](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonEKSWorkerNodePolicy.html) - Allows nodes to connect to the cluster
     - [AmazonEKS_CNI_Policy](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonEKS_CNI_Policy.html) - Allows nodes on the cluster to communicate with each other and get IP addresses
     - [AmazonEC2ContainerRegistryReadOnly](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonEC2ContainerRegistryReadOnly.html) - Allows nodes to download images from ECR
     - [AmazonEBSCSIDriverPolicy](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonEBSCSIDriverPolicy.html) - Allows cluster to use StorageClass to dynamiaclly allocate EBS
   
     The **Nodes Module** also includes addons that get created on the cluster:
       -   aws-ebs-csi-driver - A Kubernetes Container Storage Interface (CSI) plugin that provides Amazon EBS storage for your cluster.
       -   kube-proxy - Maintains network rules on each Amazon EC2 node. It enables network communication to your Pods.
       -   coredns - A flexible, extensible DNS server that can serve as the Kubernetes cluster DNS. 
       -   vpc-cni - A [Kubernetes container network interface (CNI) plugin](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) that provides native VPC networking for your cluster.

As you can see, we have addons and we have polocies to allow our nodes to use those addons, all of these except the CSI Driver are installed automatiaclly on the cluster by AWS but I still included them in the code for demonstration

  -  **ArgoCD Module** - This module deploys ArgoCD onto the cluster on launch and is pointed to our GitOps repo using `bootstrap-app.yaml` so when we run ```Terraform apply``` the infarstructue goes up with ArgoCD already running and all the charts on the GitOps repo also running, it also creates two kuberentes secrets using AWS Secret Manager, an SSH key for the GitOps repo and the MongoDB database info

