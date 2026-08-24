# Question

The Nautilus DevOps Team has received a new request from the Development Team to set up a new EC2 instance. This instance will be used to host a new application that requires a stable IP address. To ensure that the instance has a consistent public IP, an Elastic IP address needs to be associated with it. The instance will be named datacenter-ec2, and the Elastic IP will be named datacenter-eip. This setup will help the Development Team to have a reliable and consistent access point for their application.
Create an EC2 instance named `datacenter-ec2` using any linux AMI like ubuntu, the Instance type must be `t2.micro` and associate an `Elastic IP` address with this instance, name it as `datacenter-eip`.

# Step-by-Step Solution

## Option 1: Using the AWS Management Console

### 1. Launch the EC2 Instance:
**EC2 Console.**
- Log in to the AWS Management Console and navigate to EC2 > Instances.
- Click **Launch instance**.
- Under **Name and tags**, enter `datacenter-ec2`.
- Under **Application and OS Images (AMI)**, select any Linux OS (e.g., Ubuntu or Amazon Linux).
- Under **Instance type**, select `t2.micro`.
- Select or create a key pair and configure network settings as needed.
- Click **Launch instance**.

### 2. Allocate Elastic IP:
**Network & Security.**
- In the left navigation pane under **Network & Security**, click **Elastic IPs**.
- Click **Allocate Elastic IP address**.
- Under **Tags**, click **Add tag**, set **Key** to **Name** and **Value** to `datacenter-eip`.
- Click **Allocate**.

### 3. Associate Elastic IP to Instance:
**Elastic IPs.**
- Select the newly allocated Elastic IP (`datacenter-eip`).
- Click **Actions** > **Associate Elastic IP address**.
- Set **Resource type** to **Instance**.
- Select `datacenter-ec2` from the **Instance** dropdown menu.
- Click **Associate**.

## Option 2: Using the AWS CLI
Execute the following commands to launch the instance, allocate the Elastic IP, and associate them:

```Bash
# 1. Obtain an Ubuntu or Amazon Linux AMI ID (adjust region if needed)
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --query "Images[0].ImageId" \
  --output text)

# 2. Launch the datacenter-ec2 instance
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datacenter-ec2}]' \
  --query "Instances[0].InstanceId" \
  --output text)

# 3. Wait for the instance to reach running state
aws ec2 wait instance-running --instance-ids $INSTANCE_ID

# 4. Allocate Elastic IP with tag Name=datacenter-eip
ALLOCATION_ID=$(aws ec2 allocate-address \
  --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=datacenter-eip}]' \
  --query "AllocationId" \
  --output text)

# 5. Associate the Elastic IP with the instance
aws ec2 associate-address \
  --instance-id $INSTANCE_ID \
  --allocation-id $ALLOCATION_ID
```

### Verification

To verify that the setup is complete: 

- **AWS Console**: Navigate to EC2 > Instances, select datacenter-ec2, and check the Public IPv4 address field under the Details tab to confirm it displays the allocated Elastic IP address.

- **AWS CLI**: 
```bash

aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].[InstanceId, PublicIpAddress]" \
  --output table
```