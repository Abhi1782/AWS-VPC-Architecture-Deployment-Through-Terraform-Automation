# 📘 AWS-VPC-Architecture-Deployment-Through-Terraform-Automation

# 📄 Project Overview
This project covers the complete workflow for setting up AWS infrastructure automation with Terraform on an Ubuntu EC2 instance hosted in AWS.

## 🔵 Step 1 — Launch Ubuntu EC2 Server
  Using an Ubuntu EC2 instance on AWS

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (570)" src="https://github.com/user-attachments/assets/675bbff8-9eef-4723-89d8-2ac570ffd9b2" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🛠 Instructions
    
  1) Go to EC2 → Launch Instance
  2) 💻 AMI: Ubuntu Server 22.04 LTS
  3) ⚙ Instance Type: t2.micro
  4) 🔐 Key Pair: Create or use existing
  5) 🌐 Network: Default VPC
  6) 🛡 Security Group: Allow SSH (22) from your IP
  7) ▶ Click Launch

### ➤ Connect via SSH
     ssh -i your-key.pem ubuntu@<EC2-PUBLIC-IP>

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (571)" src="https://github.com/user-attachments/assets/52ef3d6c-41bf-4c45-9f23-0ff0aae5c2f0" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🟣 Step 2 — Create IAM User for Terraform

  1) Go to IAM → Users → Create User
  2) 👤 User Name: terraform-user
  3) 🔑 Select: Access Key – Programmatic Access
  4) 🛡 Permissions: AdministratorAccess (for demo only)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (573)" src="https://github.com/user-attachments/assets/d5d2d948-ed1e-42ef-9260-959cd6a07e91" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🟠 Step 3 — Generate Access Key & Secret Key

 From the IAM user:
 1) Copy/download:
    
        🔑 Access Key ID
        🕵️ Secret Access Key
    
 3) Keep these safe — used for AWS authentication in Terraform

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (574)" src="https://github.com/user-attachments/assets/d6b611c7-1df0-47cf-b93d-5452395c3910" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🟢 Step 4 — Install Terraform on Ubuntu

   📦 Install dependencies

    sudo apt-get update
    sudo apt-get install -y gnupg software-properties-common curl

## 📥 Add Terraform repository

    curl -fsSL https://apt.releases.hashicorp.com/gpg | \
    sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
    https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
    sudo tee /etc/apt/sources.list.d/hashicorp.list

## ⚙ Install

    sudo apt-get update
    sudo apt-get install terraform -y

## ✔ Verify

    terraform -version

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (572)" src="https://github.com/user-attachments/assets/55995d93-a184-4ece-a24e-3ae13d7889e4" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🔵 Step 5 — Configure AWS CLI on EC2
    
    sudo apt install awscli -y

### Configure credentials

    aws configure

Enter:
A) Access Key
B) Secret Key
C) Region (ap-south-1)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🟣 Step 6 — Create Terraform Project Directory

    mkdir terraform-vpc-project
    cd terraform-vpc-project

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

##  🟠 Step 7 — Create Terraform Files

### 📄 provider.tf

    provider "aws" {
      region = "ap-south-1"
    }

### 📄 variables.tf

    variable "aws_region" {
      description = "AWS region where resources will be created"
      type        = string
      default     = "ap-south-1"
    }
    
    variable "project_name" {
      description = "Name prefix used for tagging resources"
      type        = string
    }
    
    variable "vpc_cidr" {
      description = "CIDR block for the VPC"
      type        = string
    }
    
    variable "public_subnet_cidr" {
      description = "CIDR block for the public subnet"
      type        = string
    }

### 📄 main.tf

    # -----------------------------
    # VPC
    # -----------------------------
    resource "aws_vpc" "main" {
      cidr_block           = var.vpc_cidr
      enable_dns_support   = true
      enable_dns_hostnames = true
    
      tags = {
        Name = "${var.project_name}-vpc"
      }
    }
    
    # -----------------------------
    # Internet Gateway
    # -----------------------------
    resource "aws_internet_gateway" "igw" {
      vpc_id = aws_vpc.main.id
    
      tags = {
        Name = "${var.project_name}-igw"
      }
    }
    
    # -----------------------------
    # Public Subnet
    # -----------------------------
    resource "aws_subnet" "public" {
      vpc_id                  = aws_vpc.main.id
      cidr_block              = var.public_subnet_cidr
      map_public_ip_on_launch = true
      availability_zone       = "${var.aws_region}a"
    
      tags = {
        Name = "${var.project_name}-public-subnet"
      }
    }
    
    # -----------------------------
    # Public Route Table
    # -----------------------------
    resource "aws_route_table" "public_rt" {
      vpc_id = aws_vpc.main.id
    
      tags = {
        Name = "${var.project_name}-public-rt"
      }
    }
    
    # Route from Public RT -> Internet via IGW
    resource "aws_route" "public_internet_route" {
      route_table_id         = aws_route_table.public_rt.id
      destination_cidr_block = "0.0.0.0/0"
      gateway_id             = aws_internet_gateway.igw.id
    }
    
    # Associate Public Subnet with Public Route Table
    resource "aws_route_table_association" "public_assoc" {
      subnet_id      = aws_subnet.public.id
      route_table_id = aws_route_table.public_rt.id
    }


## 📄 outputs.tf

    output "vpc_id" {
      description = "ID of the created VPC"
      value       = aws_vpc.main.id
    }
    
    output "public_subnet_id" {
      description = "ID of the public subnet"
      value       = aws_subnet.public.id
    }
    
    output "public_route_table_id" {
      description = "ID of the public route table"
      value       = aws_route_table.public_rt.id
    }

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (575)" src="https://github.com/user-attachments/assets/769ac93e-7e77-4fcb-b50a-1f51098c5d64" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🟢 Step 8 — Initialize Terraform

    terraform init

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (576)" src="https://github.com/user-attachments/assets/16158f8e-19c4-4106-a1bc-ea5014d8815e" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (577)" src="https://github.com/user-attachments/assets/cd8156cd-ef39-48ba-858e-97bbd0146770" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## 🔵 Step 9 — Validate Terraform

    terraform validate

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1811" height="198" alt="Screenshot 2025-11-24 205527" src="https://github.com/user-attachments/assets/55e8b7b6-0c10-45dd-9e52-7935afc59d39" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🟣 Step 10 — Apply Terraform (Create VPC)

    terraform apply -auto-approve

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1907" height="594" alt="Screenshot 2025-11-24 205758" src="https://github.com/user-attachments/assets/db32e18f-f879-4d53-b43e-44df441660a8" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🟠 Step 11 — Destroy Resources (If Needed)

    terraform destroy -auto-approve

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 📤 Final Output

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (578)" src="https://github.com/user-attachments/assets/edbaba8a-9206-430e-bcff-e802eac29d15" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (579)" src="https://github.com/user-attachments/assets/7b7f5637-69a6-4800-9689-0748aadb7e7f" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (580)" src="https://github.com/user-attachments/assets/60ebfe18-a195-49a9-9a51-dcf240eab806" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img width="1920" height="1080" alt="Screenshot (581)" src="https://github.com/user-attachments/assets/bb58bb13-cf8a-4f21-ad8c-29415a51f3f8" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🎉 Project Completed

I successfully deployed an AWS VPC using Terraform automation.






