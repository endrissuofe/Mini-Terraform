# 🚀 Terraform Mini Project: EC2 Module and Security Group Module with Apache2 UserData

This mini project demonstrates how to deploy an **EC2 instance** with **Apache2** installed automatically using **Terraform modules**.  
It is for **learning purposes only** and built entirely within the AWS **Free Tier**.

---

## 📁 Project Structure

```bash
terraform-ec2-apache/
├── main.tf
├── apache_userdata.sh
└── modules/
    ├── ec2/
    │   └── main.tf
    └── security_group/
        └── main.tf
````

---

## 🧩 Step 1: Create the Project Directory

```bash
mkdir terraform-ec2-apache
cd terraform-ec2-apache
mkdir -p modules/ec2 modules/security_group
```

---

## ⚙️ Step 2: Create the Security Group Module

**File:** `modules/security_group/main.tf`

This module creates a security group allowing inbound HTTP (port 80) and SSH (port 22) traffic.

```hcl
resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow HTTP and SSH"
  vpc_id      = data.aws_vpc.default.id

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

output "sg_id" {
  value = aws_security_group.web_sg.id
}
```

---

## 🖥️ Step 3: Create the EC2 Module

**File:** `modules/ec2/main.tf`

This module launches an EC2 instance using the provided Security Group and runs a UserData script to install Apache2.

```hcl
resource "aws_instance" "web" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = "t2.micro"
  vpc_security_group_ids = [var.security_group_id]
  user_data              = file("${path.module}/../../apache_userdata.sh")

  tags = {
    Name = "apache-web-instance"
  }
}

output "public_ip" {
  value = aws_instance.web.public_ip
}
```

![github](img/ec2-instance.png)

---

## 📜 Step 4: Create the UserData Script

**File:** `apache_userdata.sh`

```bash
#!/bin/bash
sudo apt update -y
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
echo "Hello from Terraform EC2 with Apache!" | sudo tee /var/www/html/index.html
```

Make it executable:

```bash
chmod +x apache_userdata.sh
```

![github](img/userdata.png)

---

## 🧠 Step 5: Create the Main Terraform Configuration

**File:** `main.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}

data "aws_vpc" "default" {
  default = true
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

module "security_group" {
  source = "./modules/security_group"
}

module "ec2" {
  source            = "./modules/ec2"
  security_group_id = module.security_group.sg_id
}

output "ec2_public_ip" {
  value = module.ec2.public_ip
}
```

![github](img/main-tf.png)

---

## 🚀 Step 6: Initialize and Apply

```bash
terraform init
terraform apply -auto-approve
```

Once applied, Terraform will output the **public IP** of the EC2 instance.

![github](img/terraform-apply.png)

---

## 🌐 Step 7: Verify Apache Installation

Copy the public IP from the Terraform output and open it in your browser:

```
http://<public_ip>
```

You should see the message:

> **Hello from Terraform EC2 with Apache!**

![github](img/apache-homepage.png)

---

## 🧹 Step 8: Destroy Resources (to stay within Free Tier)

After testing, clean up resources to avoid charges:

```bash
terraform destroy -auto-approve
```

![github](img/terraform-destroy.png)

---

## 🧾 Observations & Lessons Learned

* Terraform modules help in **code reuse and cleaner structure**.
* UserData makes it easy to **automate software installation** on EC2 launch.
* Security Groups are essential for **controlling inbound and outbound access**.
* Always **destroy resources** after testing to avoid costs.

---
