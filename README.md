Envoy Deployment with Pod Autoscaling and Node Autoscaling on EKS
Overview
This project demonstrates how to deploy an Envoy Proxy application on Amazon EKS and implement:
    - Envoy Deployment 
    - Node Group Labeling 
    - Pod Scheduling using Node Selector 
    - Horizontal Pod Autoscaler (HPA) 
    - Cluster Autoscaler for Node Autoscaling
Architecture

                    +-------------------+
                    |   Client Traffic  |
                    +---------+---------+
                              |
                              v
                    +-------------------+
                    |   Envoy Service   |
                    +---------+---------+
                              |
                    +---------+---------+
                    |                   |
              +-----v-----+       +-----v-----+
              | Envoy Pod |       | Envoy Pod |
              +-----------+       +-----------+
                       HPA Scales Pods
                              |
                    Cluster Autoscaler
                              |
                    Adds/Removes Nodes
Prerequisites
    - AWS Account 
    - EKS Cluster 
    - kubectl configured 
    - eksctl installed 
    - AWS CLI configured 
    - IAM OIDC provider enabled 

Step 1: Create Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: envoy
Apply:
kubectl apply -f namespace.yaml
Step 2: Create Node Group with Label
Create a dedicated node group for Envoy workloads.
eksctl create nodegroup \
  --cluster dev-cluster \
  --name gtm-node \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 5 \
  --node-labels Name=your-name
Verify:

kubectl get nodes --show-labels
Step 3: Deploy Envoy Application
envoy-deployment.yaml
Step 4: Expose Envoy Service
envoy-service.yaml

Step 5: Configure Horizontal Pod Autoscaler
envoy-hpa.yaml


Step 6: Generate Load for HPA Testing
Run temporary load generator:
kubectl run load-generator \
  --rm -it \
  --image=curlimages/curl \
  -n envoy -- sh
Inside pod:
while true; do curl http://envoy-service; done
Watch scaling:
kubectl get hpa -n envoy -w
kubectl get pods -n envoy -w
Step 7: Configure Cluster Autoscaler
Associate OIDC Provider
eksctl utils associate-iam-oidc-provider \
  --cluster dev-cluster \
  --approve
Create IAM Policy

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeScalingActivities",
                "ec2:DescribeImages",
                "ec2:DescribeInstanceTypes",
                "ec2:DescribeLaunchTemplateVersions",
                "ec2:GetInstanceTypesFromInstanceRequirements",
                "eks:DescribeNodegroup"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}

Step 8: Create the IAM Role for OIDC Provider and attach the policy created

Step 9: Create IAM Service Account

Step 10: Deploy Cluster Autoscaler

Step 11: Test Node Autoscaling


Increase replicas:
kubectl scale deployment envoy-deploy \
  --replicas=20 -n envoy
Watch pods:
kubectl get pods -n envoy
