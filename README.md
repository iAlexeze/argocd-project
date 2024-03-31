
# ArgoCD CI/CD Pipeline Setup Guide

This guide provides a step-by-step approach to setting up a CI/CD pipeline using Jenkins, ArgoCD, and Kubernetes, focusing on deploying a Docker-based application named `vpro-app`.

## Prerequisites

- AWS CLI installed and configured with IAM user credentials of admin access.
- `kubectl` installed.
- `kops` installed for Kubernetes cluster management.
- A domain purchased and a public hosted zone created in AWS Route 53.
- An S3 bucket created to store the cluster's state.

## Setting Up the Environment

1. **Install AWS CLI**:

    ```bash
    sudo apt update 
    sudo apt install awscli -y
    aws configure
    ```

2. **Install kubectl**:

    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    ```

3. **Install kops**:

    ```bash
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```

4. **Create Kubernetes Cluster with kops**:

    ```bash
    kops create cluster --name=k8.learnwithabhi.xyz --state=s3://mykopsbucketfork8 --zones=us-east-1a --node-count=1 --node-size=t3.small --master-size=t3.medium --dns-zone=k8.learnwithabhi.xyz --node-volume-size=8 --master-volume-size=8
    kops update cluster --name k8.learnwithabhi.xyz --state=s3://mykopsbucketfork8 --yes --admin
    kops validate cluster --state=s3://mykopsbucketfork8
    ```

## Installing ArgoCD

1. **Create ArgoCD Namespace**:

    ```bash
    kubectl create namespace argocd
    ```

2. **Install ArgoCD**:

    ```bash
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

3. **Install ArgoCD CLI**:

    ```bash
    curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
    sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
    rm argocd-linux-amd64
    ```

4. **Configure ArgoCD Server Access**:

    ```bash
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
    ```
    You can also achieve the same thing with a Node Port Service
    ```bash
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort", "ports": [{"name": "argocd-server", "port": 80, "nodePort": 31111}]}}'

    ```

## Accessing ArgoCD

1. **Login to ArgoCD**:

    ```bash
    argocd admin initial-password -n argocd
    ```

2. **Change Password**: Update your password through the ArgoCD UI.

## Deploying the Application

1. **Clone the Git Repository**:

    ```bash
    git clone https://github.com/learnwithabhishek/argocd-project.git](https://github.com/iAlexeze/argocd-project.git
    ```

2. **Create ArgoCD Application**:

    ```bash
    kubectl apply -f application.yaml
    ```

3. **Verify Deployment**: Use the ArgoCD UI to monitor the deployment status and access details about deployments, replica sets, and pods.

4. **Access the Application**: Use the LoadBalancer external IP to access your deployed application.

## Cleaning Up

After completing your project, delete the ArgoCD application and the Kubernetes cluster to clean up resources.

```bash
kops delete cluster --name k8.learnwithabhi.xyz --state=s3://mykopsbucketfork8 --yes
```

This guide aims to provide a comprehensive setup for a CI/CD pipeline using Jenkins, ArgoCD, and Kubernetes, focusing on deploying a Docker-based application named `vpro-app`.


