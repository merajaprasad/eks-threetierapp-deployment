# ThreeTierApp Devops Project

## Project Overview
The project involves deploying a Three-Tier Web Application using ReactJS, NodeJS, and MongoDB, with deployment on AWS EKS.

[![](https://app.eraser.io/workspace/YW2RSs1i3JwonaT25oeg/preview?elements=ig4UqOgNPFl1dX15b2Vwkg&type=embed)](https://app.eraser.io/workspace/YW2RSs1i3JwonaT25oeg?elements=ig4UqOgNPFl1dX15b2Vwkg)

[![](https://app.eraser.io/workspace/YW2RSs1i3JwonaT25oeg/preview?elements=iljAd0EwRpkBU_T-iMV7Tw&type=embed)](https://app.eraser.io/workspace/YW2RSs1i3JwonaT25oeg?elements=iljAd0EwRpkBU_T-iMV7Tw)

## Application Code
The `Application-Code` directory contains the source code for the Three-Tier Web Application. Dive into this directory to explore the frontend and backend implementations.

## Kubernetes Manifests-Files
The `Kubernetes-Manifests-Files` directory holds Kubernetes manifests for deploying your application on AWS EKS. Understand and customize these files to suit your project needs.


📈 **The journey covered everything from setting up tools to deploying a Three-Tier app, ensuring data persistence, and implementing CI/CD pipelines.**

### Step 1: IAM Configuration
- Create a user `eks-admin` with `AdministratorAccess`.
- Generate AWS Security Credentials: Access Key and Secret Access Key.

### Step 2: EC2 Setup
- Launch an Ubuntu instance and do SSH into the instance from your local machine.
  
### Clone the Repositoty**
```bash
git clone <repository-URL>
```

### Step 3: Install Docker
``` shell
sudo apt-get update -y
sudo apt install docker.io -y
docker --version
docker ps
sudo chown $USER /var/run/docker.sock
```
### Step 4: Install and Configure AWS CLI v2
``` shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

## Setup Frontend
go inside frontand directory `/cd Application-Code/frontend` and create a `Dockerfile` using below code.
```
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```
**Build Docker image and run container**
```
docker build -t three-tier-frontend .
docker run -d -p 3000:3000 three-tier-frontend:latest
```
now you can see our website using [publicip : 3000 ]

## Setup Backend
now go inside frontand directory `/cd Application-Code/backend` and create a `Dockerfile` using below code.
```
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "index.js"]
```
**build Docker image and run container**
```
docker build -t three-tier-backend .
docker run -d -p 3500:3500 three-tier-backend:latest
docker logs
```
Now push all the images to Docker HUB one after one.

## Kubernertes Configuration

### Step 1: Install kubectl
``` shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Step 2: Install eksctl
``` shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 3: Setup EKS Cluster
``` shell
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```

## Run Manifests Files
### create namespace
``` shell
kubectl create namespace workshop
```
### Create and run databse
got inside `Kubernetes-Manifests-file/Database` and run below command
```
kubectl apply -f .

kubectl delete -f .
```
### Create and run backend
got inside `Kubernetes-Manifests-file/Backend` and run below command
```
kubectl apply -f .

kubectl delete -f .
kubectl logs <podname> -n workshop          # database connection check
```
### Create and run frontend
got inside `Kubernetes-Manifests-file/Frontend` and run below command
```
kubectl apply -f .
kubectl delete -f .
kubectl get pods -n workshop
```

## Loadbalancer Setup with EKS
### Step 1: IAM Service Account Setup

**Download LB IAM policy**
``` shell
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
**Create IAM policy**

this policy help to connect EKS and Loadbalancer with each other
```shell
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicyForEKS --policy-document file://iam_policy.json
```
**Create OIDC Provider**
```shell
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
```
**Create IAM Service Account**

IAM Service account help to communicate between services (EKS and ALB)
```shell
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::975050304823:policy/AWSLoadBalancerControllerIAMPolicyForEKS --approve --region=us-west-2
```

### Step 2: Deploy AWS Load Balancer Controller inside EKS-Cluster
**Install Helm**
``` shell
sudo snap install helm --classic
```
**Install and Add helm-Charts for EKS**
```shell
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm repo list
```
**Install Load Balancer Controller inside kube-system**
```shell
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=three-tier-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```
**Varify LB controller status**
```shell
kubectl get deployment -n kube-system aws-load-balancer-controller
```
**Apply ingress**
```shell
kubectl apply -f ingress.yaml
kubectl get ing -n workshop
```
**Domain Registration**
after running ```kubectl get ing -n workshop``` command we will recieved load balancer DNS Name. then copy this load balancer name and create a CNAME Record in your Domain Provider. 

### Cleanup
- To delete the EKS cluster:
``` shell
eksctl delete cluster --name three-tier-cluster --region us-west-2
```

## Contribution Guidelines
- Fork the repository and create your feature branch.
- Deploy the application, adding your creative enhancements.
- Submit a Pull Request with a detailed description of your changes.

## Support and collaboration
Thanks to [Shubham Londhe](https://www.trainwithshubham.com/) helps to Create this Project. For any queries, please open an issue in the repository.

...

