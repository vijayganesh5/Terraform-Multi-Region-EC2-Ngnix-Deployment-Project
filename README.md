# 🌍 Terraform Multi-Region EC2 + Nginx Deployment

## 🧠 Project Overview

This project demonstrates how to **launch two AWS EC2 instances** in **two different regions** (for example: Mumbai and N. Virginia) using a **single Terraform file**.
Each EC2 instance will automatically have **Nginx installed and started**, all set up from your **AWS EC2 Ubuntu instance** using Terraform 🚀

---

## ⚙️ Tech Stack Used

| Tool              | Purpose                                             |
| ----------------- | --------------------------------------------------- |
| ☁️ **AWS EC2**    | For creating cloud instances                        |
| 🏗️ **Terraform** | Infrastructure automation tool                      |
| 💻 **AWS CLI**    | For authentication and credentials setup            |
| 🐧 **Ubuntu**     | Used as the base instance to run Terraform commands |

---

## 🧩 Prerequisites

Before starting, make sure you have:

1. ✅ **AWS EC2 Ubuntu instance** to work on
2. ⚙️ **Terraform installed**

   ```bash
   sudo apt update -y
   sudo apt install unzip -y
   wget https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_linux_amd64.zip
   unzip terraform_1.6.6_linux_amd64.zip
   sudo mv terraform /usr/local/bin/
   terraform -version
   ```
3. 🔑 **AWS CLI configured**

   ```bash
   aws configure
   ```

   Enter:

   * AWS Access Key ID
   * AWS Secret Access Key
   * Default region (e.g., ap-south-1)
   * Output format: json
4. 🧾 **IAM User** with EC2 and VPC permissions

---

## 🪜 Steps to Perform

### 1️⃣ Create Project Directory

```bash
mkdir terraform-nginx-multi-region
cd terraform-nginx-multi-region
```

---

### 2️⃣ Create Terraform Configuration File

Create a file named **main.tf** and paste the following:

```hcl
provider "aws" {
  region = "ap-south-1"
}

provider "aws" {
  alias  = "us"
  region = "us-east-1"
}

# Security Group allowing SSH & HTTP
resource "aws_security_group" "nginx_sg" {
  name        = "nginx-sg"
  description = "Allow SSH and HTTP inbound traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
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
}

# EC2 Instance in India Region
resource "aws_instance" "india_ec2" {
  ami           = "ami-0dee22c13ea7a9a67"
  instance_type = "t2.micro"
  security_groups = [aws_security_group.nginx_sg.name]
  tags = {
    Name = "India-Nginx"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt update -y",
      "sudo apt install nginx -y",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx"
    ]
  }

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
}

# EC2 Instance in US Region
resource "aws_instance" "us_ec2" {
  provider      = aws.us
  ami           = "ami-0866a3c8686eaeeba"
  instance_type = "t2.micro"
  security_groups = [aws_security_group.nginx_sg.name]
  tags = {
    Name = "US-Nginx"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt update -y",
      "sudo apt install nginx -y",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx"
    ]
  }

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
}
```

---

### 3️⃣ Initialize Terraform

```bash
terraform init
```

---

### 4️⃣ Validate Terraform File

```bash
terraform validate
```

---

### 5️⃣ Plan the Infrastructure

```bash
terraform plan
```

---

### 6️⃣ Deploy the Infrastructure

```bash
terraform apply -auto-approve
```

Terraform will:
✅ Launch EC2 instance in **ap-south-1 (India)**
✅ Launch EC2 instance in **us-east-1 (US)**
✅ Install **Nginx** automatically on both

---

## 🔍 Verification

1. Go to **AWS EC2 Console** in both regions
2. Check that both EC2 instances are running
3. Copy the **Public IP** of each instance
4. Open them in your browser:

   ```
   http://<PUBLIC-IP>
   ```

   You should see the **Nginx default welcome page** 🎉

---

## 🧹 Cleanup

When done, destroy all resources to avoid extra AWS charges 💰

```bash
terraform destroy -auto-approve
```

---

## 🧾 Summary

| Action                          | Description                                         |
| ------------------------------- | --------------------------------------------------- |
| 🌍 **Deployed 2 EC2 Instances** | One in India (ap-south-1) and one in US (us-east-1) |
| 🧰 **Used Terraform**           | Automated multi-region infrastructure setup         |
| 🌐 **Installed Nginx**          | Verified with public IPs                            |
| 🧹 **Cleaned up**               | Safely removed all created resources                |

---

## 🎯 Final Outcome

You have successfully automated the creation of **two EC2 instances in different AWS regions**, each pre-configured with **Nginx** using **Terraform** 🏗️✨

Your infrastructure is now **multi-region, reproducible, and easily destroyable** — just like a true DevOps Engineer! 😎

---

## 🎉 Output Link with Screenshots:
- https://docs.google.com/document/d/1O2xYrzzj1psTHgV33PL-JM2tc131imxPhwG2nSzRKp4/edit?usp=sharing

---

