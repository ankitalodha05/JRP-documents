Here's a **well-structured and beginner-friendly document** for the provided Terraform code. It explains the setup of a **VPC and EKS Cluster** using Terraform modules.

---

# 📘 Terraform Setup for AWS VPC and EKS Cluster (`nareshit-cluster`)

## 🔰 Objective

Provision a fully functional **Amazon EKS (Elastic Kubernetes Service) Cluster** along with a **custom VPC** using reusable Terraform modules.

---

## 🧾 Prerequisites

* Terraform CLI installed
* AWS CLI configured (`aws configure`)
* IAM permissions to create VPC, EKS, Subnets, NAT Gateway, etc.
* Basic understanding of Terraform modules and AWS networking

---

## 🛠️ Code Breakdown

### 1. **Provider Configuration**

```hcl
provider "aws" {
  region = local.region
}
```

* Uses the AWS provider.
* Region is dynamically picked from `local.region`.

---

### 2. **Local Variables**

```hcl
locals {
  name   = "nareshit-cluster"
  region = "us-east-1"

  vpc_cidr = "10.123.0.0/16"
  azs      = ["us-east-1a", "us-east-1b"]

  public_subnets  = ["10.123.1.0/24", "10.123.2.0/24"]
  private_subnets = ["10.123.3.0/24", "10.123.4.0/24"]
  intra_subnets   = ["10.123.5.0/24", "10.123.6.0/24"]

  tags = {
    Example = local.name
  }
}
```

* Defines project name, region, and subnets (public, private, intra).
* Tags are set for resource identification.

---

### 3. **VPC Module**

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 4.0"

  name = local.name
  cidr = local.vpc_cidr

  azs             = local.azs
  private_subnets = local.private_subnets
  public_subnets  = local.public_subnets
  intra_subnets   = local.intra_subnets

  enable_nat_gateway = true

  public_subnet_tags = {
    "kubernetes.io/role/elb" = 1
  }

  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = 1
  }
}
```

* Uses the official `terraform-aws-modules/vpc/aws` module.
* Creates:

  * VPC with `10.123.0.0/16` CIDR
  * 2x Public Subnets
  * 2x Private Subnets
  * 2x Intra Subnets (used for EKS Control Plane)
* NAT Gateway enabled for internet access to private subnets.
* Subnets are tagged for use with EKS.

---

### 4. **EKS Cluster Module**

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.15.1"

  cluster_name                   = local.name
  cluster_endpoint_public_access = true

  cluster_addons = {
    coredns    = { most_recent = true }
    kube-proxy = { most_recent = true }
    vpc-cni    = { most_recent = true }
  }

  vpc_id                   = module.vpc.vpc_id
  subnet_ids               = module.vpc.private_subnets
  control_plane_subnet_ids = module.vpc.intra_subnets

  eks_managed_node_group_defaults = {
    ami_type       = "AL2_x86_64"
    instance_types = ["t2.small"]
    attach_cluster_primary_security_group = true
  }

  eks_managed_node_groups = {
    ascode-cluster-wg = {
      min_size     = 1
      max_size     = 2
      desired_size = 1
      instance_types = ["t2.small"]
      capacity_type  = "SPOT"
      tags = {
        ExtraTag = "helloworld"
      }
    }
  }

  tags = local.tags
}
```

* Uses the `terraform-aws-modules/eks/aws` module.
* EKS Cluster named `nareshit-cluster` with public endpoint.
* Adds three essential EKS addons:

  * `CoreDNS`
  * `kube-proxy`
  * `VPC CNI`
* EKS control plane placed in intra subnets.
* Private subnets used for worker nodes.
* One managed node group:

  * Spot instances (`t2.small`)
  * Autoscaling: min=1, max=2
  * Tagged with `ExtraTag = helloworld`

---

## 🧪 How to Deploy

### 🔹 Step 1: Initialize Terraform

```bash
terraform init
```

### 🔹 Step 2: Validate the Configuration

```bash
terraform validate
```

### 🔹 Step 3: Review the Execution Plan

```bash
terraform plan
```

### 🔹 Step 4: Apply the Changes

```bash
terraform apply
```

---

## 📎 Output Resources

After deployment, you will get:

* A new VPC with 6 subnets (2x each type)
* NAT Gateway for internet access in private subnets
* EKS cluster with default addons
* One managed node group with autoscaling (SPOT instances)

---

## 📌 Notes

* Spot instances reduce cost but may be interrupted.
* You can later update `eks_managed_node_groups` to add on-demand or GPU-based instances.
* Ensure your IAM user/role has required permissions for EKS, VPC, EC2, IAM, and related services.

---

Would you like a `.md` version of this document or want to extend this setup (e.g., with ALB Ingress, ArgoCD, monitoring tools)?
