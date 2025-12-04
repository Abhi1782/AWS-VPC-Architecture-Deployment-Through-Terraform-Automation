# 📘 AWS-VPC-Architecture-Deployment-Through-Terraform-Automation

# 📄 Project Overview
This project covers the complete workflow for setting up AWS infrastructure automation with Terraform on an Ubuntu EC2 instance hosted in AWS.

## 🔵 Step 1 — Launch Ubuntu EC2 Server
  Using an Ubuntu EC2 instance on AWS

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

## 🟣 Step 2 — Create IAM User for Terraform

  1) Go to IAM → Users → Create User
  2) 👤 User Name: terraform-user
  3) 🔑 Select: Access Key – Programmatic Access
  4) 🛡 Permissions: AdministratorAccess (for demo only)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🟠 Step 3 — Generate Access Key & Secret Key

 From the IAM user:
 1) Copy/download:
    
        🔑 Access Key ID
        🕵️ Secret Access Key
    
 3) Keep these safe — used for AWS authentication in Terraform

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


