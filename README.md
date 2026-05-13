**Mind Track Application Deployment – DevOps CI/CD Project**

**Project Overview**

This project demonstrates a complete end-to-end DevOps deployment workflow for the Mind Track application using AWS cloud-native services and Kubernetes orchestration.

The application repository contained only the production-ready dist/ directory, which represents the final compiled frontend build. Since the project already included the built application artifacts, no Node.js build process or package installation was required.

The primary objective of this project was to implement a production-style CI/CD pipeline using Docker, Amazon ECR, Amazon EKS, AWS CodeBuild, AWS CodePipeline, and Kubernetes.
--------------------------------------------------------------
**Project Environment
AWS Services Used**

Amazon EC2
Amazon ECR
Amazon EKS
AWS CodeBuild
AWS CodePipeline
AWS CloudWatch
IAM

**Tools and Technologies**

Docker
Kubernetes
kubectl
eksctl
GitHub
Nginx
AWS CLI
**
Application Repository**

Original Repository Provided:

**Brain-Tasks-App Repository**

The repository contained:

**dist/
README.md**

Since the repository already included the production-ready build files, no package.json or application source code was required.

**Step 1 – EC2 Instance Setup**

An Ubuntu EC2 instance was launched to configure the DevOps environment.

Instance Configuration
Configuration	Value
Operating System	Ubuntu
Instance Type	t3.small
Region	us-east-1

Inbound security group rules configured:

Port 22 – SSH
Port 80 – HTTP
Port 443 – HTTPS
Port 3000 – Application Testing

**Step 2 – DevOps Tools Installation**

The following tools were installed and configured inside the EC2 instance:

**Docker Installation**

sudo apt install docker.io -y

Docker service was started and enabled successfully.

AWS CLI Installation

AWS CLI v2 was installed and configured using IAM user credentials.

kubectl Installation

kubectl was installed for Kubernetes cluster management.

eksctl Installation

eksctl was installed for Amazon EKS cluster creation and management.

**Step 3 – Repository Cloning**

The application repository was cloned inside the EC2 instance.

git clone https://github.com/Vennilavanguvi/Brain-Tasks-App.git

**Step 4 – Docker Containerization**

A Dockerfile was manually created to containerize the application using Nginx.

Dockerfile

FROM public.ecr.aws/nginx/nginx:alpine

COPY dist/ /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

**Step 5 – Local Docker Validation**

The Docker image was built locally using:

docker build -t brain-tasks-app .

The container was started successfully:

docker run -d -p 3000:80 --name brain-tasks-container brain-tasks-app

The application was verified successfully using the EC2 public IP:

**http://54.173.90.19:3000**

**Step 6 – Amazon ECR Setup**

An Amazon ECR private repository was created.

Repository Name
brain-tasks-app

The Docker image was tagged and pushed successfully into Amazon ECR.

Docker Tag Command
docker tag brain-tasks-app:latest 691850985444.dkr.ecr.us-east-1.amazonaws.com/brain-tasks-app:latest
Docker Push Command
docker push 691850985444.dkr.ecr.us-east-1.amazonaws.com/brain-tasks-app:latest

**Step 7 – Amazon EKS Cluster Creation**

An Amazon EKS cluster was created using eksctl.

Cluster Configuration
Configuration	Value
Cluster Name	trend-cluster
Region	us-east-1
Node Type	t3.small
Node Count	2
Cluster Creation Command

eksctl create cluster \
--name trend-cluster \
--region us-east-1 \
--nodes 2 \
--node-type t3.small \
--managed

Cluster nodes were verified successfully using:

kubectl get nodes

**Step 8 – Kubernetes Deployment**

**deployment.yaml**

A Kubernetes deployment configuration file was created manually to deploy the application using the ECR Docker image.

The deployment configuration included:

Replica configuration
Container image from ECR
Port exposure
Kubernetes labels
service.yaml

A Kubernetes LoadBalancer service was created to expose the application externally.

type: LoadBalancer

The application was successfully exposed using the AWS LoadBalancer URL.

**Step 9 – AWS CodeBuild Integration**

AWS CodeBuild was configured to automate:

Docker image build
Docker image tagging
Docker image push to ECR
Deployment to Amazon EKS
buildspec.yml Configuration

The buildspec.yml file was manually configured with Docker build and Kubernetes deployment automation.

Docker Build Process
docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
Docker Tag Process
docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
Docker Push Process
docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
EKS Deployment Process
aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name $CLUSTER_NAME
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl rollout restart deployment brain-tasks-app

This implementation confirmed that Docker image build, push, and EKS deployment were fully automated using AWS CodeBuild.

**Step 10 – IAM Role Mapping for EKS Access**

The AWS CodeBuild service role was granted Kubernetes administrative access using IAM identity mapping.

IAM Identity Mapping Command
eksctl create iamidentitymapping \
--cluster trend-cluster \
--region us-east-1 \
--arn arn:aws:iam::691850985444:role/codebuild-brain-tasks-build-service-role \
--group system:masters \
--username admin

This allowed CodeBuild to execute kubectl commands inside the EKS cluster.

**Step 11 – AWS CodePipeline Automation**

AWS CodePipeline was configured for complete CI/CD automation.

Pipeline Flow
GitHub
   ↓
AWS CodePipeline
   ↓
AWS CodeBuild
   ↓
Docker Build
   ↓
Amazon ECR
   ↓
Amazon EKS
   ↓
Kubernetes Deployment
----------------------------------------------
Whenever code is pushed to GitHub:

CodePipeline automatically detects changes
CodeBuild starts the build process
Docker image is built automatically
Docker image is pushed to ECR
Kubernetes deployment is updated inside EKS

**Step 12 – CloudWatch Monitoring**

AWS CloudWatch Logs were used for monitoring:

Docker build logs
Kubernetes deployment logs
ECR push logs
Build failures
CI/CD execution tracking

CodeBuild execution logs were successfully monitored through CloudWatch Log Groups.

Kubernetes Verification
Verify Kubernetes Pods
kubectl get pods
Verify Kubernetes Services
kubectl get svc
Verify Kubernetes Deployments
kubectl get deployment

All Kubernetes resources were verified successfully and were in running state.

**Final Project Outcome**

The Mind Track application was successfully:

Containerized using Docker
Stored in Amazon ECR
Deployed on Amazon EKS
Automated using AWS CodeBuild
Integrated with AWS CodePipeline
Exposed using Kubernetes LoadBalancer
Monitored using AWS CloudWatch

The final deployment architecture implemented a complete CI/CD automation pipeline from GitHub to Kubernetes deployment on Amazon EKS.
