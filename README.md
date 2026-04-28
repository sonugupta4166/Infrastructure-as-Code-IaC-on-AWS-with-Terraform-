# 🚀 AWS Infrastructure as Code (IaC) with Terraform

![Terraform](https://img.shields.io/badge/Terraform-IaC-623CE4?style=for-the-badge&logo=terraform)
![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?style=for-the-badge&logo=amazonaws)
![GitHub](https://img.shields.io/badge/GitHub-Version_Control-181717?style=for-the-badge&logo=github)
![Infrastructure](https://img.shields.io/badge/Infrastructure-Automation-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Project-Production_Style-success?style=for-the-badge)

---

# 📌 Project Overview

This project demonstrates **production-style AWS Infrastructure as Code (IaC)** using **Terraform** to provision secure, scalable cloud infrastructure.

It includes networking, compute, storage, IAM, and Terraform remote backend concepts aligned with DevOps best practices.

---

# 🏗️ Infrastructure Components

## 🌐 Networking
- ✅ Custom VPC
- ✅ Public Subnet
- ✅ Private Subnet
- ✅ Internet Gateway
- ✅ Route Table
- ✅ Route Table Association

## 🔐 Security
- ✅ Security Group (SSH + HTTP)
- ✅ IAM Role for EC2

## 💻 Compute
- ✅ EC2 Instance

## 📦 Storage
- ✅ S3 Bucket
- ✅ Remote State Ready
- ✅ DynamoDB Locking Ready

---

# 🖼️ Architecture Diagram

```text
                    🌍 Internet
                        │
                        ▼
               🚪 Internet Gateway
                        │
                        ▼
              🌐 Custom AWS VPC (10.0.0.0/16)
                   ┌───────────────┐
                   │               │
                   ▼               ▼
        🌍 Public Subnet      🔒 Private Subnet
         (10.0.1.0/24)         (10.0.2.0/24)
               │
               ▼
         💻 EC2 Instance
               │
               ▼
         📦 S3 Bucket

⚙️ provider.tf
terraform {
  required_version = ">= 1.0"

  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "infra/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-lock-table"
    encrypt        = true
  }
}

provider "aws" {
  region = var.aws_region
}

🧩 variables.tf
variable "aws_region" {
  default = "ap-south-1"
}

variable "vpc_cidr" {
  default = "10.0.0.0/16"
}

variable "public_subnet_cidr" {
  default = "10.0.1.0/24"
}

variable "private_subnet_cidr" {
  default = "10.0.2.0/24"
}

variable "instance_type" {
  default = "t2.micro"
}

variable "ami_id" {
  description = "Amazon Linux 2 AMI"
  default     = "ami-0f5ee92e2d63afc18"
}

variable "bucket_name" {
  default = "your-unique-project-bucket"
}

🛠️ terraform.tfvars
aws_region  = "ap-south-1"
bucket_name = "your-unique-project-bucket"

🏗️ main.tf
# VPC
resource "aws_vpc" "main_vpc" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "main-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id

  tags = {
    Name = "main-igw"
  }
}

# Public Subnet
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = var.public_subnet_cidr
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet"
  }
}

# Private Subnet
resource "aws_subnet" "private_subnet" {
  vpc_id     = aws_vpc.main_vpc.id
  cidr_block = var.private_subnet_cidr

  tags = {
    Name = "private-subnet"
  }
}

# Public Route Table
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "public-route-table"
  }
}

# Route Table Association
resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}

# Security Group
resource "aws_security_group" "ec2_sg" {
  vpc_id = aws_vpc.main_vpc.id

  ingress {
    description = "SSH Access"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTP Access"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "ec2-security-group"
  }
}

# IAM Role
resource "aws_iam_role" "ec2_role" {
  name = "ec2_s3_access_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

# EC2 Instance
resource "aws_instance" "web_server" {
  ami                         = var.ami_id
  instance_type               = var.instance_type
  subnet_id                   = aws_subnet.public_subnet.id
  vpc_security_group_ids      = [aws_security_group.ec2_sg.id]
  associate_public_ip_address = true

  tags = {
    Name = "terraform-web-server"
  }
}

# S3 Bucket
resource "aws_s3_bucket" "project_bucket" {
  bucket = var.bucket_name

  tags = {
    Name = "project-s3-bucket"
  }
}

📤 outputs.tf
output "vpc_id" {
  value = aws_vpc.main_vpc.id
}

output "public_subnet_id" {
  value = aws_subnet.public_subnet.id
}

output "private_subnet_id" {
  value = aws_subnet.private_subnet.id
}

output "ec2_public_ip" {
  value = aws_instance.web_server.public_ip
}

output "s3_bucket_name" {
  value = aws_s3_bucket.project_bucket.bucket
}

🚫 .gitignore
.terraform/
*.tfstate
*.tfstate.backup
terraform.tfvars

🚀 Deployment Commands
terraform init
terraform validate
terraform plan
terraform apply

🌟 Future Enhancements
🔄 GitHub Actions CI/CD Pipeline
📦 Ansible Configuration Automation
⚖️ Load Balancer Integration
📈 Auto Scaling Group
☸️ Kubernetes (EKS)
📊 Monitoring with CloudWatch
