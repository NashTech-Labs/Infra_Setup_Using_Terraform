# Terraform Infrastructure as Code (IaC) for AWS

## Introduction

This repository contains Terraform code to provision infrastructure on AWS using Infrastructure as Code (IaC) principles. With Terraform, you can define and manage your AWS resources in a declarative manner.

## Getting Started

To use this Terraform configuration, follow these steps:

1. Install Terraform on your local machine.
2. Clone this repository to your local machine.

## Configuration

The provided Terraform script provisions the following resources on AWS:

### AWS Provider Configuration

```hcl
provider "aws" {
  region = "us-west-2" # Change to your desired AWS region
}
```

### Virtual Private Cloud (VPC) and Subnet

```hcl
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "private_subnet" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-west-2a" # Change to your desired availability zone
}
```

### Amazon Elastic Kubernetes Service (EKS) Cluster

```hcl
resource "aws_eks_cluster" "my_cluster" {
  name     = "my-eks-cluster"
  role_arn = "arn:aws:iam::123456789012:role/eks-service-role" # Change to your EKS service role ARN
  version  = "1.21" # Change to your desired EKS version

  vpc_config {
    subnet_ids = [aws_subnet.private_subnet.id]
  }
}
```

### EKS Worker Nodes

```hcl
resource "aws_eks_node_group" "my_node_group" {
  cluster_name     = aws_eks_cluster.my_cluster.name
  node_group_name  = "my-node-group"
  node_role_arn    = "arn:aws:iam::123456789012:role/eks-node-role" # Change to your EKS node role ARN
  subnet_ids       = [aws_subnet.private_subnet.id]
  instance_types   = ["t2.medium"]
  desired_capacity = 3
}
```

### EC2 Instances

```hcl
resource "aws_instance" "my_instances" {
  count         = 4
  ami           = "ami-0c55b159cbfafe1f0" # Change to your desired AMI
  instance_type = "t2.medium"
  user_data     = <<-EOF
     #!/bin/bash
     sudo apt update
     chmod 700 ~/.ssh
     chmod 600 ~/.ssh/authorized_keys
  EOF

  tags = {
    Name = "my-instance-${count.index + 1}"
  }
}
```

### SSH Key Configuration

```hcl
resource "null_resource" "copy_ssh_key" {
  depends_on = [aws_instance.my_instances]

  provisioner "remote-exec" {
    connection {
      type        = "ssh"
      host        = aws_instance.my_instances.*.public_ip[count.index]
      user        = "ubuntu" # Change to your desired username
      private_key = file("~/.ssh/your_private_key.pem") # Change to your private key path
    }
    inline = [
      "echo 'YOUR_PUBLIC_KEY' >> ~/.ssh/authorized_keys" # Change to your public key
    ]
  }
}
```

## Usage

To apply this Terraform configuration:

1. Navigate to the cloned repository directory.
2. Initialize Terraform: `terraform init`.
3. Review the execution plan: `terraform plan`.
4. Apply the configuration: `terraform apply`.

Make sure to replace placeholders like region, availability zone, AMI, IAM role ARNs, and public/private key paths with your actual configurations. Additionally, ensure that you have the necessary IAM roles and policies attached to your AWS account for EKS cluster creation and EC2 instance provisioning.

