# LetsReview GitOps (STILL A WORK IN PROGRESS)
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

## Phase 0: The App
First I wrote a simple flask application with a single HTML file [LetsReview-App](https://github.com/adigaandyt/LetsReview-App).
The app has a few API calls and is doing CRUD operations with a MongoDB database

## Phase 1: Repository Setup 
I set up 3 repositories for this project,
1) [LetsReview-App](https://github.com/adigaandyt/LetsReview-App) The application repo which contains
    - The flask app we're deploying
    - Dockerfile to create an image of the app
    - Docker-compose file to create a local environment for tests, with a diagram
    - Jenkinsfile for the CI pipeline
    - Test script for an E2E test for all the API calls


2) [LetsReview-Infra](https://github.com/adigaandyt/LetsReview-Infra)
