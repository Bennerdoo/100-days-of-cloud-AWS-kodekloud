# Question

The Nautilus DevOps Team is working on setting up a new web server for a critical application. The team lead has requested you to create an EC2 instance that will serve as a web server using Nginx. This instance will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.

As a member of the Nautilus DevOps Team, your task is to create an EC2 instance with the following specifications:

Instance Name: The EC2 instance must be named `                      xfusion-ec2`.

AMI: Use any available Ubuntu AMI to create this instance.

User Data Script: Configure the instance to run a user data script during its launch. This script should:

Install the Nginx package.

Start the Nginx service.

Security Group: Ensure that the instance allows HTTP traffic on port 80 from the internet.

# Step-by-step Solution

## Option 1: Using the AWS Management Console

### 1. Launch the EC2 Instance:

**EC2 Console:**
Open the EC2 Console and click Launch instance. Set the Name tag to `xfusion-ec2`. Under Application and OS Images (AMI), select an appropriate Ubuntu AMI (e.g., Ubuntu 22.04 LTS). Choose an instance type (e.g., `t2.micro`), configure networking and storage as needed, and ensure a security group allows HTTP (port 80) traffic. Review and click Launch instance.

### 2. Configure User Data Script:

**Advanced details:**
In the Launch instance wizard, expand Advanced details. In the User data text area, paste the following script to install and start Nginx:

```Bash
#!/bin/bash
apt-get update -y
apt-get install nginx -y
systemctl start nginx
systemctl enable nginx
```

Click Next until you reach the Review and launch page. Review your settings and click Launch instance.

### 3. Verify Nginx Installation:

**Instance status and verification:**
After the instance launches, copy its Public IPv4 address from the EC2 Console. Open a web browser and navigate to `http://<Public_IP_Address>`. You should see the default Nginx welcome page.

## Option 2: Using the AWS CLI

Run the following AWS CLI script to automate the entire process:

```Bash

# 1. Retrieve the default VPC ID
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text)

# 2. Create Security Group allowing Port 80
SG_ID=$(aws ec2 create-security-group \
  --group-name xfusion-web-sg \
  --description "Allow HTTP traffic on port 80" \
  --vpc-id $VPC_ID \
  --query "GroupId" \
  --output text)

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# 3. Retrieve latest Ubuntu 22.04 AMI ID
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --query "Images[0].ImageId" \
  --output text)

# 4. Define User Data script (base64 encoded automatically or via inline execution)
USER_DATA=$(cat <<'EOF'
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl start nginx
systemctl enable nginx
EOF
)

# 5. Launch the xfusion-ec2 instance
aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --security-group-ids $SG_ID \
  --user-data "$USER_DATA" \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]'
```

### 3. Verify Nginx Installation:

Get the public IP address of the newly created xfusion-ec2 instance:

```Bash


aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" "Name=instance-state-name,Values=running" \
  --query "Reservations[0].Instances[0].PublicIpAddress" \
  --output text
```

Open your web browser or run curl against the instance's public IP:

```Bash


curl -I http://<PUBLIC_IP>
```

A successful response returns HTTP/1.1 200 OK serving the Welcome to Nginx landing page.