# Our Library GitOps 
This repository contains the GitOps configuration for the Our_Library WebApp, using Argo CD to manage deployments to an EKS cluster. The configuration defines the desired state of our applications and infrastructure, ensuring that our deployment environment is declarative, version-controlled, and automated.
and is currently hosted [here](http://ourlibargo.ddns.net/) for demonstration purposes
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

