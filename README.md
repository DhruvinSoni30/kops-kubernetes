# How to Set up Kubernetes Cluster Using Kops on AWS?

Kops is one of the tools to create and manage the Kubernetes cluster. Kops stands for Kubernetes operations. Using kops you can create the production-ready highly available Kubernetes cluster. Using kops you can create kubernetes cluster in many cloud providers.

Kops is responsible for the entire lifecycle of the cluster, from infrastructure provisioning to upgrading to deleting, and it knows about everything about worker nodes, master nodes, load balancers, cloud providers, monitoring, networking, logging, etc.

![kops](https://github.com/DhruvinSoni30/Images/blob/main/kops.jpeg?raw=true)

<h1>Features of Kops:</h1>

* Create a highly available cluster
* Command-line auto-completion
* A state-sync model for dry-runs and automatic idempotency
* YAML Manifest Based API Configuration
* Support for custom Kubernetes add-ons

Kops uses AWS CLI commands in order to create the resources on the AWS cloud. Kpos will create Auto scaling groups & launch configurations for both master & worker nodes through which the whole cluster remains highly available.

Auto-scaling groups will ensure that we have enough numbers of master and worker nodes.

<h1>Prerequisite:</h1>

* AWS Account
* Basic understanding of Kubernetes and docker

Now, let's start with the cluster provisioning using Kops on AWS

**Step 1:- Create Kops Server**
For creating the cluster we will first create a kops server. Kops server will respond to interact with master and worker nodes. Through the Kops server, we will manage the entire cluster.

Instance details:
![instance](https://github.com/DhruvinSoni30/Images/blob/main/server.png?raw=true)

Kops server is responsible to create other resources (i.e. master & worker node etc) so, in order to create them, we need to assign the IAM role to the Kops server.

**1.1 Create 1 IAM role with the below policies and attach it to the Kops server.**

* AmazonEC2FullAcces
* AmazonVPCFullAccess
* IAMFulllAccess
* AmazonS3FullAccess

**1.2** In this Kops server, we need to install **AWS CLI, Kops tool & kubectl.**

**1.3** Run below aws configureand provide the details to configure **AWS CLI.**

**Step 2:- Create an S3 bucket**

Kops server needs to store the cluster configurations and state details somewhere. So, we need to create 1 S3 bucket so, that the Kops server can store the information.

Run this command from the Kops server to create the S3 bucket.
`aws s3 mb s3://<bucket-name>`

**Step 3:- Expose the environment variables**

**3.1** Add the below code to the **.bashrc** file

`vi .bashrc`

`export NAME=<name of cluster>.k8s.local (k8s.local is the must part)`

`export KOPS_STATE_STORE=s3://<bucket-name>`

**3.2** Run below command

`source .bashrc`

**Step 4:- Create SSH keys**

In order to SSH into the master & worker nodes, we need to create the SSH keys so, in order to create it run the below command

`ssh-keygen`

**Step 5:- Create the cluster definition**

In order to create the cluster definition run the below command.

**Note:-** Below command will just create the cluster definition and store it in S3. It will not create the actual cluster.

`kops create cluster --zones us-east-1a --networking weave --master-size t2.medium --master-count 1 --node-size t2.micro --node-count=2 ${NAME}`

**Note:-**
1. You can give multiple zones (comma separately) for high availability.
2. We are using weavenet kubernetes network
3. Make sure to use t2.medium or higher size for the master

**Step 6:-** Create a secret for cluster

Run the below command to create the secret for the nodes so, that we can SSH into it.

`kops create secret - name ${NAME} sshpublickey admin -i ~/.ssh/<key-name>.pub`

**Step 7:- Create the actual cluster**

So, now our definition file is ready so, we can run the below command to create the cluster

`kops update cluster ${NAME} --yes`

**Step 8:- Validate the cluster**

The cluster creation can take up to 20 mins to set up everything meanwhile we can validate it by running the below command.

`kops validate cluster`

**Note:-** If you are getting this `Validation failed: unexpected error during validation: error listing nodes: Unauthorized` error then run the below command to resolve it.

`kops export kubecfg - admin`

Kops will create the below resources in order to set up and manage the cluster.

* Launch Templates for master & worker nodes (It has the user data for configuration machines)
* Auto-scaling groups for master & worker nodes
* 1 Master node & 2 worker nodes
* Load balancer (Pointing to the master node/API-server)

**Step 9:-** Create an application

Now our kubernetes cluster is ready. So, it's time to create an application.

The file provided at above. Use that file and run below command to deploy the application.

`kubectl apply -f website.yaml`

The above code will create a deployment & service of the type load balancer.

Run the below command to check the pods and service.

`kubectl get pods` This command will print the pod's details. 

`kubectl get svc` This command will print the service's details.

Run the below command in order to describe the service

`kubectl describe svc website`

The above command will give you some output and copy the value of LoadBalancer ingress from it and paste it in the web browser you will webpage like below.

![home page](https://github.com/DhruvinSoni30/Images/blob/main/website.png?raw=true)




