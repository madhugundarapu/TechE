
# Assignment: 
## Automate EC2 Deployment 
## Requirements#
## Checklist: 

Task                                        Status 
AWS Free Tier Sing Up                        yes 
Launch EC2 of specific type                  yes 
Install Java21 & Maven                       yes 
Clone & Deploy GitHub App                    yes 
Run app via java -jar                        yes 
App runs Port 80, /hello returns text        yes 
Auto-stop EC2 after delay                    yes 
Use env for AWS keys(not hardcoded)          yes 
Stage-based config loading (Dev, Prod)       yes

### Folder Structure

### css 
ec2-deployment/ 
──> main.tf 
├──> variables.tf 
├──> outputs.tf 
├──> user_data.sh 
├──> configs/ 
│ 
|     ├──> dev_config
|     └──> prod_config

## Step-by-step Breakdown 

## 1 main.tf -EC2 Launch via Terraform 
### hcl 
provider "aws" { 
  region = var.aws_region
}

resource "aws_instance" "devops_ec2" { 
  ami = var.ami_id 
  instance_type = var.instance_type 
  key_name = var.kay_name 
  user_data = templatefile("user_data.sh", { 
          stage = var.stage 
})

tags = {
  Name = "EC2-${var.stage}"
  }
} 
  output "ec2_public_ip" { value = aws_instance.devops_ec2.public_ip 
}

## 2. variables.tf 
### hcl 
 variable "aws_region" { 
             default = "us-east-1" 
}
  variable "instance_type" { 
            default = "t2.micro" 
}

variable "ami_id" { 
            default = "ami-xxxxxxxxxxxx" # Amazon Linux 2 
}

variable "key_name" { 
            description = "Your existing AWS key pair" 
            default = "your-key-name" }

variable "stage" { 
            description = "Deployment stage: Dev or Prod" 
            default = "Dev" }

## 3. user_data.sh (Uses templatefile variables) 
### bash

#!/bin/bash 
exec > /var/log/user-data.log 2>&1 
echo "====== USER DATA START ======"

### Update system
sudo yum update -yes

### Install Java 21
sudo amazon-linux-extras enable java-openjdk21 
sudo yum Install-y java-21-openjdk java-21-openjdk-devel

### Install Maven
wget https: //downloads.apache.org/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz tar -xvzf apache-maven-3.9.6-bin.tar.gz 
sudo mv apache-maven-3.9.6 /opt/Maven

echo 'export M2_HOME=/opt/maven' | sudo tee -a /etc/profile echo 'export PATH=$M2_HOME/bin:$PATH | sudo tee -a /etc/profile 
source /etc/profile

### Install Git 
sudo yum install -y Git

### Clone GitHub Repo
cd /home/ec2-USER git clone https: // github.com/Trainings-TechEazy/test-repo-for-devops 
cd test-repo-for-devops

### Copy config file based on stage
cp ../configs/${stage}_config ./config.properties

### Build project 
/opt/maven/bin/mvn clean package

### Allow HTTP on port 80 
sudo yum install -y nginx 
sudo systemctl start nginx 
sudo systemctl enable nginx 
sudo firewall-cmd --add-port=80/tcp --permanent 
sudo firewall-cmd --reload

### Run Spring Boot app on port 80 
nohup java -jar target/hellomvc-0.0.1-SNAPSHOT.jar --server.port=80

### Schedule shutdown after 60 minutes
sudo shutdown -h +60

echo "====== USER DATA END ======"

## 4. configs/dev_config (example)
### ini

env=Dev db_url=jdbc:mysql://dev-db-url

## 5. configs/prod_config(example)
### ini

env=Prod db_url=jdbc:mysql://prod-db-url

### Testing

after terraform apply, get the public IP:

### bash

https:///hello

### you should see
### csharp

Hello from Spring MVC!

### Security Notes

DO NOT hardcode access keys. Expoet AWS keys in terminal session only:

### bash
export AWS_ACCESS_KEY_ID="your_key" 
export AWS_SECRET_ACCESS_KEY="your_secret"

### Clean Up
stop the instance manually (or it's already scheduled to stop in 60 min):

### bash

terraform destroy
