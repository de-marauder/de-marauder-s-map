---
title: "Multi-region Kubernetes Cluster deployments with Terragrunt"
seoTitle: "Multi-region Kubernetes deployments"
seoDescription: "Multi-region deployments can be done with Terragrunt to improve maintainability and configuration DRYness"
datePublished: Sun Jun 16 2024 11:07:30 GMT+0000 (Coordinated Universal Time)
cuid: clxhfzrgk000409jq5mfaco4w
slug: multi-region-kubernetes-cluster-deployments-with-terragrunt
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1718533311232/cf5afa02-046d-4c7f-bc65-da1b5ea0045c.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1718535817005/2bfba87c-1fbb-4ec6-a863-7e311b6acae9.png
tags: kubernetes, terraform, eks, gitops, argocd, terragrunt

---

There are many reasons to create a multi-region infrastructure. You might have users distributed across the world and aim to decrease latency by bringing services closer to them (similar to the function of CDNs). It could also be for disaster recovery or for implementing canary or rollout deployment strategies. Whatever the case, this article provides a detailed guide on how to set up your Infrastructure as Code (IaC) declarations cleanly and efficiently.

## Pre-requisites

* **An AWS account**: Please be informed about the cost implications as the free tier will not cover some of the services used.
    
* Terraform
    
* Terragrunt
    

# Introduction

Terragrunt is an Infrastructure as Code (IaC) tool. It is a thin wrapper around Terraform that helps you write Terraform configurations using a module-first approach and a DRY (Don't Repeat Yourself) methodology, leading to minimal duplication. Due to these advantages, it is being preferred over Terraform for this project.

Before we begin, you’ll want to visit the Terragrunt [site](https://terragrunt.gruntwork.io/docs/getting-started/install/) for official installation instructions.

At any point in the article, you can refer to this [GitHub repo](https://github.com/de-marauder/multi-region-eks) for the complete project.  

# Infrastructure Plan

For this demo, we will demonstrate how to deploy EKS (Elastic Kubernetes Service) clusters to multiple environments and regions in AWS. We will also incorporate a GitOps approach while designing this architecture.

GitOps is the practice of storing and versioning your infrastructure declarations in one or more repositories that serve as the primary source of truth for the desired state of your infrastructure. All changes to the infrastructure state are made through these repositories, ensuring continuous synchronization between the actual state and the declared state.

Different architectures can be adopted depending on the tool used to implement your GitOps workflow. For this exercise, we will use ArgoCD as our GitOps tool. There are two major architectural plans we could adopt:

1. **Centralized Push Method**: We have a single standalone cluster to run our GitOps services, pushing changes to our infrastructure (in this case, edge clusters).
    

![](https://lh7-us.googleusercontent.com/docsz/AD_4nXcv7FG23ovpvbFkwJxMITlEXON-p8sEB2fZZUgiQdXwAfRpOPQmXQ_s2p1vyshAdLnFPQoLkUjze27saWQ0ppClhp-j3LSP7LqKqX-RI5EITWiaGP8WEX0GMobIuEMRe7tLJUg1axiuwLLcAVz5rEHPytc?key=LdXAeXoCs3pTZRubdSP2rA align="left")

2. **Distributed Pull Method**: All clusters in our infrastructure run a dedicated GitOps agent that independently pulls changes.
    

![](https://lh7-us.googleusercontent.com/docsz/AD_4nXdOZAhXayWdXfFoN0BQ7aXtrUmc7hsyyNkxS3ayiQ45NzvE6OB3hVXvsThE9TKXAOJwxafXEKETz90p0D0zzOPaSC9cgHlezIeO3uSQzGaPhYRnWHctjClfO_Tsh0w-DTzCpKkd7_f7HgqBGVxYV-hDdIdy?key=LdXAeXoCs3pTZRubdSP2rA align="left")

The centralized push method introduces a single point of failure but streamlines access to the clusters, allowing for easy monitoring and observability. This would be quite cumbersome for the distributed approach to handle.

## Architecture - Hub and Spoke

The architecture chosen for this exercise is the Hub and Spoke architecture, which is based on the centralized push model. For simplicity, we will focus on deploying a single cluster per environment per region. However, this can be scaled to multiple clusters with minimal changes.

![](https://lh7-us.googleusercontent.com/docsz/AD_4nXdo3Qd3IwYZRUD0zV6oBc9vIRQ-jbOkuRusQLE747i8Yh0XAsD1BCsfKu6c4aau9uL8rPuDyx06Xlx1ce3wWqQ-xxb6yYqS8x_0iurwAM-889TkuWC1ImKrPG3Re_36C17sL8nfo0Enu7GskxJGV1nwWs4?key=LdXAeXoCs3pTZRubdSP2rA align="left")

## What we’ll need

To create the best system, we need to outline and consider all components properly.

First, we need a VPC (Virtual Private Cloud) per cluster per region. This is necessary for security and because AWS VPCs are region-specific.

Next, we’ll deploy our clusters. EKS (Elastic Kubernetes Service) is a managed Kubernetes service that simplifies cluster setup and management. To enhance security, clusters will be hidden from the public internet. EKS integrates with AWS IAM for authorization and authentication, which we will utilize. We need one GitOps cluster and edge clusters in each desired region.

Finally, the GitOps cluster must communicate with all other clusters. Since VPCs do not span regions by default, we need to bridge them. AWS provides solutions like Transit Gateways (TGW) and VPC Peering, which are ideal for this setup.

Transit Gateway (TGW) provides an infinitely scalable connection point for VPC networks. A single TGW can connect multiple VPCs. VPC peering, on the other hand, provides a one-to-one connection between two VPCs, either within the same region or across different regions. You can think of them as similar to virtual Ethernet (veth) peers in the Linux networking stack.

It's important to note that a TGW is restricted to a region and can only communicate cross-region via peering connections to other TGWs in their respective regions. Essentially, you can think of the TGW as a router that can be bridged to another router to extend a network.

![](https://lh7-us.googleusercontent.com/docsz/AD_4nXd_NFoyRRpDhmf1aW4tRCYgzEwavqJUZKztLk9pRcKDgKMANUQgz6M0KM1_jP3UvvdkqhwR37cKGgNhUq4CgZDNu6bxwTzNI6pApwV89skikfqz7B38PRWfyo4gyc0qXdVBXSE3iK01kgFz7n3ey_82_C0x?key=LdXAeXoCs3pTZRubdSP2rA align="left")

Due to cost implications and the fact that we are only deploying a single cluster and VPC per region, we will opt for VPC peering connections to provide communication links between the GitOps cluster and the edge clusters in our infrastructure.

Now, let's begin!

# Terragrunt setup

Once again, all the code for this project can be found [here](https://github.com/de-marauder/multi-region-eks). 

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This is not a Terragrunt tutorial. We will only cover as much as is required to understand the project. If you’d like to know more please check out their <a target="_blank" rel="noopener noreferrer nofollow" href="https://terragrunt.gruntwork.io/docs/" style="pointer-events: none">official docs</a>.</div>
</div>

We start by defining our directory structure with ease of management in mind. The structure we’ll be using looks like this

```bash
terragrunt/
├── backend.tf
├── environments
│   ├── _env/
│   ├── development
│   │   ├── env.hcl
│   │   ├── regions
│   │   │   ├── us-east-2/
│   │   │   └── us-west-2/
│   │   └── shared
│   └── production
├── env.yaml
├── modules/
└── terragrunt.hcl
```

Our entry point is a terragrunt.hcl file, with possible configurations injected via env.yaml at the root. We also define our Terraform modules. Remember, Terragrunt is a superset of Terraform, so valid Terraform code remains valid in Terragrunt. We then define our environments (e.g., development, production, staging). In each environment, we define shared resources (resources used across regions) and specify the distinct regions where we want to deploy our components.

At this point, you might be thinking, “Do I have to copy over the region and environment configs every time I want to add a new region or environment?” Unfortunately, the answer is yes. As of now, there is no cleaner way to provide infrastructure for multiple regions, at least according to my findings. Correct me if I'm wrong.

You might also wonder, “Can’t we just loop over these resources using something like a list of regions?” Again, the answer is no, at least not for the entire configuration. Some resources that need to be provisioned in each region require provider information specific to that region, and Terraform and Terragrunt do not support looping over resources that accept a provider as an argument.

> A module intended to be called by one or more other modules must not contain any provider blocks. A module containing its own provider configurations is not compatible with the for\_each, count, and depends\_on arguments that were introduced in Terraform v0.13.

Source: [https://developer.hashicorp.com/terraform/language/modules/develop/providers](https://developer.hashicorp.com/terraform/language/modules/develop/providers)

## Relevant Modules

We will now discuss the various modules developed for our infrastructure configurations and the considerations behind them.

### IAM Module

This module creates a role and policy for accessing the EKS cluster. Amazon EKS uses IAM protocols like roles and policies to ensure secure access to your clusters. It does this using a feature called access entries, which is set to replace the old method of identity management on EKS, the aws-auth ConfigMap. While preparing your cluster, it’s advised to create dedicated roles that will be used to access the cluster. These roles can then be attached to EKS access policies to grant different IAM principals access to the cluster and its resources.

```bash
resource "aws_iam_role" "eks_cluster_role" {

 name = "${local.project_name}-role"

 assume_role_policy = jsonencode({
   "Version" : "2012-10-17",
   "Statement" : [
     {
       "Effect" : "Allow",
       "Principal" : {
         "Service" : "eks.amazonaws.com"
         # // You can add iam user arns to attach roles to them
         # // This will allow them to assume this role
         # # "AWS" : "arn:aws:iam::xxxxxxxxx:user/xxxxxxxxx"
         "AWS" : "${var.principal}"
       },
       "Action" : "sts:AssumeRole"
     }
   ]
 })

 tags = local.tags
}

data "aws_iam_policy" "eks_cluster_policy" {
 arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
 tags = local.tags
}
```

### EKS Module

This is based on the official EKS terraform module provided by AWS. It is configured to be private by default. It has its own VPC and security groups to restrict access to it to only selected CIDR ranges. 

The IAM roles defined are used to define access entries so you can define the different access rights users have on your cluster.

```bash
module "eks" {
 source  = "terraform-aws-modules/eks/aws"
 version = "~> 20.0"

# Other configurations are omitted

 access_entries = {
   # One access entry with a policy associated
   admin = {
     kubernetes_groups = []
     principal_arn     = "${var.iam_role_arn}"
     policy_associations = {
       eks_admin = {
         policy_arn = "arn:aws:eks::aws:cluster-access-policy/AmazonEKSAdminPolicy"
         access_scope = {
           type = "cluster"
         }
       }

       eks_cluster_admin = {
         policy_arn = "arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy"
         access_scope = {
           type = "cluster"
         }
       }
     }
}
```

There are 6 EKS cluster access policies and you can select any combination of them to attach to your roles in your access entries. Check out the [docs](https://docs.aws.amazon.com/eks/latest/userguide/access-policies.html#access-policy-permissions) for more information 

```bash
aws eks list-access-policies
{
         "accessPolicies": [
             {
                 "name": "AmazonEKSAdminPolicy",
                 "arn": "arn:aws:eks::aws:cluster-access-policy/AmazonEKSAdminPolicy"
             },
             {
                 "name": "AmazonEKSAdminViewPolicy",
                 "arn": "arn:aws:eks::aws:cluster-access-policy/AmazonEKSAdminViewPolicy"
             },
             {
                 "name": "AmazonEKSClusterAdminPolicy",
                 "arn": "arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy"
             },
             {
                 "name": "AmazonEKSEditPolicy",
                 "arn": "arn:aws:eks::aws:cluster-access-policy/AmazonEKSEditPolicy"
             },
             {
                 "name": "AmazonEKSViewPolicy",
                 "arn": "arn:aws:eks::aws:cluster-access-policy/AmazonEKSViewPolicy"
             },
             {
                 "name": "AmazonEMRJobPolicy",
                 "arn": "arn:aws:eks::aws:cluster-access-policy/AmazonEMRJobPolicy"
             }
         ]
}
```

### GitOps Cluster Module

This module builds on the previously defined EKS module, adding a bastion host to the VPC as the single point of entry into our cluster network. The bastion host is set up using `user_data` to install necessary binaries like `kubectl`, `argocd`, `helm`, and `aws-cli`. The user\_data also configures initial settings and starts services like the ArgoCD GUI, accessible via an ingress application load balancer. Security groups are implemented to block unwanted access.

### VPC Peering Requester Module

This module creates a VPC peer connection from one VPC to another essentially extending a VPC’s network (kind of like an ethernet cable or Veth pair). Since our architecture involves cross-region communication, a request to connect has to be sent to the peering region.

### VPC Peering Accepter Module

This module contains configurations to accept peer connections automatically from requesting regions using the peer connection ID. Once the connection is accepted, the two VPCs can talk to each other.

### EBS-addon Module

This module uses an OpenID Connect (OIDC) provider to provision roles and policies, granting a service account the privileges to create an EBS CSI (Elastic Block Storage Container Storage Interface) driver. This allows the cluster to handle persistentVolumeClaims and create persistentVolumes using EBS as the storage backend, which is useful for applications like Consul or databases. This feature is optionally added to the EKS module configuration.

## Bringing Everything Together

To complete our setup, we use Terragrunt. First, we create the entry point `terragrunt.hcl` file and define configurations for our state backend and default providers. Terragrunt can automatically create an S3 remote backend if the specified one doesn’t exist. 

Built with a module-first and DRY approach, Terragrunt offers several features to reduce code duplication, such as the include block for importing external configurations and the dependency block for determining module execution order and passing outputs between modules.

With these features in mind, we can start creating our Terragrunt modules. Each module references a Terraform module via a source path (relative or absolute) and provides inputs for the module.

We can also define dependencies and include blocks to reference external Terragrunt configurations, allowing us to share common settings between different modules. For example, edge clusters in each region could share module input variables or provider configurations. We use the include block to include the root terragrunt config in all our modules.

```bash
# terragrunt module configuration showing how include blocks are used and module sources are passed.

include "root" {
 path = find_in_parent_folders()
}

include "env" {
 path   = "${get_terragrunt_dir()}/../../../../../environments/_env/edge-cluster.hcl"
 expose = true
}

terraform {
 source = "${include.env.locals.source_base_url}"
}
```

In our setup, there’s an `_env` directory to store module configurations reusable across environments.

```bash
│   ├── _env
│   │   ├── edge-cluster.hcl
│   │   ├── gitops-cluster.hcl
│   │   ├── vpc-peering-conn-accepter.hcl
│   │   └── vpc-peering-conn-requester.hcl
```

This helps to keep the main terragrunt module configuration files lean and our entire setup DRY.

We also have configuration files in the regions directories and environment directories to share region-specific (region.hcl) and environment-specific (env.hcl) parameters.

```bash
├── environments
│   ├── development
│   │   ├── env.hcl
│   │   ├── regions/
│   │   │   ├── us-east-2
│   │   │   │   ├── region.hcl
```

For this setup, we just have some local variables defined which we read into modules requiring them.

```bash
# region.hcl
locals {
 region = "us-east-2"
}
```

Reading these files is done like so,

```bash
locals {
 environment_var = read_terragrunt_config(find_in_parent_folders("env.hcl"))
 region_var      = read_terragrunt_config(find_in_parent_folders("region.hcl"))

 env_name = "${local.environment_var.locals.environment}"
 region   = "${local.region_var.locals.region}"
}
```

Our shared directory is made up of the GitOps cluster and IAM Terragrunt module configurations. 

```bash
│   │   └── shared
│   │       ├── gitops-cluster
│   │       │   ├── secrets.tfvars
│   │       │   └── terragrunt.hcl
│   │       └── iam
│   │           ├── secrets.tfvars
│   │           └── terragrunt.hcl
```

Secrets are also passed using `.tfvars` files (make sure to not commit these files to git).

Each region is made up of an edge cluster, a VPC peer connection requester, and a VPC peer connection accepter.

```bash
│   │   ├── regions
│   │   │   ├── us-east-2
│   │   │   │   ├── edge-cluster
│   │   │   │   │   └── terragrunt.hcl
│   │   │   │   ├── region.hcl
│   │   │   │   ├── vpc-peer-connection-accepter
│   │   │   │   │   └── terragrunt.hcl
│   │   │   │   └── vpc-peering-requester
│   │   │   │       └── terragrunt.hcl
```

With this, our setup is pretty much complete and can now be scaled by copy-pasting the required folders.

## Running Our Configuration

To run the configurations, you can navigate to the root of the project and run

```bash
terragrunt run-all plan
terragrunt run-all apply
terragrunt run-all destroy
```

The `run-all` subcommand tells terragrunt to run all the submodules in the project. A terragrunt module is indicated by a directory with a `terragrunt.hcl` file in it.

# Conclusion

In this article, we’ve seen how one might approach setting up a multi-cluster, multi-region deployment. We’ve evaluated some considerations that should be taken while doing this. The key takeaways are as follows:

* Optimize for reusability. For this purpose, Terragrunt was favored over plain terraform because it helps make your configurations DRYer.
    
* Select an appropriate directory structure to manage everything efficiently.
    

The entire codebase can be found [here](https://github.com/de-marauder/multi-region-eks) for you to scan at your leisure.

I’d also like to hear your thoughts on this topic. How are you currently deploying your multi-region infrastructure? Let me know in the comments or via my social handles.

Till we meet again….Cheers!

![Create meme: memes, meme happy birthday, Leonardo DiCaprio](https://imgs.search.brave.com/aIeSTlTysQuJFlF_NqOUztUZEjVypWy1vaOdOyGCt6I/rs:fit:860:0:0/g:ce/aHR0cHM6Ly93d3cu/bWVtZS1hcnNlbmFs/LmNvbS9tZW1lcy80/ODI5NTFiZGY5MjMw/OGY5ZWVlMWExNGFm/ZjMzODU0OS5qcGc align="left")