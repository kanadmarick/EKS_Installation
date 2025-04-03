# EKS Installation Guide with Ingress for App Deployment

## Project Overview
This step-by-step guide explains how to install Amazon Elastic Kubernetes Service (EKS) using AWS, including ingress functions for application deployment.

## Pre-requisites
Ensure you have the following tools installed before proceeding:

- **kubectl** – A command-line tool for managing Kubernetes clusters. [Installation Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- **eksctl** – A command-line tool for automating EKS cluster operations. [Installation Guide](https://eksctl.io/introduction/installation/)
- **AWS CLI** – A command-line tool for interacting with AWS services, including EKS. [Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

## AWS Fargate Overview
AWS Fargate is a serverless compute engine that runs containers without requiring users to manage the underlying infrastructure. It works with both Amazon Elastic Container Service (ECS) and Amazon Elastic Kubernetes Service (EKS). Key benefits include:

- No need to provision or manage servers.
- Seamless integration with ECS and EKS.
- Automatic scaling and patching.

## Installation Steps

1. **Configure AWS CLI**
   ```sh
   aws configure
   ```
   Provide your AWS access key, secret key, region, and default output format.

2. **Create an EKS Cluster**
   ```sh
   eksctl create cluster --name my-cluster --region us-west-2 --nodegroup-name my-nodes --node-type t3.medium --nodes 3
   ```

3. **Verify Cluster Status**
   ```sh
   kubectl get svc
   ```

4. **Deploy an Application with Ingress**
   - Install Nginx Ingress Controller:
     ```sh
     kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/aws/deploy.yaml
     ```
   - Create an Ingress Resource:
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: my-app-ingress
     spec:
       rules:
       - host: my-app.example.com
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: my-app-service
                 port:
                   number: 80
     ```
   - Apply the Ingress configuration:
     ```sh
     kubectl apply -f my-ingress.yaml
     ```
5. **Create fargate profile**
```sh
eksctl create fargateprofile --cluster demo-cluster-1 --region us-east-1 --name alb-sample-app --namespace game-2048
```

6. **Deploy ALB (Application Load Balancer)**

   - Connect IAM OIDC provider to the cluster:
     ```sh
     eksctl utils associate-iam-oidc-provider --cluster demo-cluster-1 --approve
     ```

7. **Create ALB IAM Policy as provided by ALB Controller**
   - Download the policy JSON file:
     ```sh
     curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
     ```
   - Create IAM Policy:
     ```sh
     aws iam create-policy \
         --policy-name AWSLoadBalancerControllerIAMPolicy \
         --policy-document file://iam_policy.json
     ```
   - Create IAM Role and attach it to the service account:
     ```sh
     eksctl create iamserviceaccount \
       --cluster=<your-cluster-name> \
       --namespace=kube-system \
       --name=aws-load-balancer-controller \
       --role-name AmazonEKSLoadBalancerControllerRole \
       --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
       --approve
     ```

8. **Install Helm Chart for AWS Load Balancer Controller**
   - Install Helm:
     ```sh
     curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
     echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
     sudo apt-get update
     sudo apt-get install helm
     helm version
     ```
   - Add Helm repo and install AWS Load Balancer Controller:
     ```sh
     helm repo add eks https://aws.github.io/eks-charts
     helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
       -n kube-system \
       --set clusterName=<your-cluster-name> \
       --set serviceAccount.create=false \
       --set serviceAccount.name=aws-load-balancer-controller \
       --set region=<region> \
       --set vpcId=<your-vpc-id>
     ```
![image](https://github.com/user-attachments/assets/b36a3c8d-a287-4f1a-9154-cc2cead34c11)

9. **Wait for Load Balancer to come online**
   - Copy the DNS name of the load balancer and access your application from a web browser.
  
   - ![image](https://github.com/user-attachments/assets/ecf818b9-0e4d-413e-b707-555a1451c288)


![image](https://github.com/user-attachments/assets/3b64757d-d12c-4f98-af3d-a2de0f5ea801)

![image](https://github.com/user-attachments/assets/99053fb2-281a-403b-b3b7-ee3edfb619e1)

## Next Steps
- Configure DNS for your application.
- Implement security policies and IAM roles.
- Monitor and scale your EKS cluster as needed.
