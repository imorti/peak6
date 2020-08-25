# Overview
The purpose of this document is to describe technical and design decisions regarding the cloud deployment of this application. 

# Cloud provider and Infrastructure
Using Terraform, we'll establish an infrastructure repo to utilize IaC (Infrastructure as Code). We'll set up an AWS EKS (Elastic Kubernetes Service) cluster. This will be multi-node running in us-east-1 region across 3 availability zones. The control plane will consist of two api servers across two availability zones to ensure high availability. A separate repo will be used for infrastructure manipulation versus applications. INgress

## Components
For ingress well use the EKS ALB Ingress controller. We'll utilize a Cluster Autoscaler for scalability of the infrastructure. The metrics server and HPA (Horizontal Pod Autoscaler) will be responsible for dealing with pods scaling within the cluster. RDS will be used to store transactional data. Route53 will be used to manage DNS records. Elastic Container Registry (ECR) will be used to store the different containers and versions with incremental tagging. AWS Certificate Manager will be used for securing HTTPS on applications. Cloudtrail can be used for traceability into API calls and environment updates. For monitoring we can use Prometheus and Grafana for data collection and visualization. 

# CI/CD
Due to it's prevalence in the industry, JenkinsX will be used for CI/CD. It's the preferred CI tool given any devops engineer would have had to come to know it. Separate pipelines will be used for applications versus infrastructure. Utilizing pull requests, we can also create preview environments using namespaces names after pull requests (Ex. PR-35-AppName). Updates to Slack channel can notify teams of successful deployments. 

# Deployment to target environments
For deployment to different environments, we'll use branches in Github repos mapping to environments. For each application, there will be a dev, int, uat and prod branch. Upon a `git push` to the target branch, it will use a web hook to JenkinsX to pull the latest code, run a build, create a container, push to ECR, then declaratively inform the EKS cluster to run the properly tagged container from ECR. 

The dev, int, uat environments will utilize one cluster and different namespaces in the cluster while the prod environment will be on a separate VPC, cluster and infrastructure. This will address cost and utilization concerns. 


# Current status
Container is stored on my dockerhub at https://hub.docker.com/repository/docker/imorti/peak6