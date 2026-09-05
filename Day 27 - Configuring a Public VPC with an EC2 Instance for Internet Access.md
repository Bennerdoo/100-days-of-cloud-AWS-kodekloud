# Question

The Nautilus DevOps Team has received a request from the Networking Team to set up a new public VPC to support a set of public-facing services. This VPC will host various resources that need to be accessible over the internet. As part of this setup, you need to ensure the VPC has public subnets with automatic IP assignment for resources. Additionally, a new EC2 instance will be launched within this VPC to host public applications that require SSH access. This setup will enable the Networking Team to deploy and manage public-facing applications.

Create a public VPC named `xfusion-pub-vpc`, and a subnet named `xfusion-pub-subnet` under the same, make sure public IP is being auto assigned to resources under this subnet. Further, create an EC2 instance named `xfusion-pub-ec2` under this VPC with instance type t2.micro. Make sure SSH port 22 is open for this instance and accessible over the internet.

# Step by step solution

## Option 1: Using the AWS Management Console

### Create VPC, Subnet, and Internet Gateway

***VPC Console***
Open the VPC Console and click Create VPC.

Choose VPC and more (or create manually):

Name tag: `xfusion-pub-vpc`

IPv4 CIDR block: `10.0.0.0/16`

Number of public subnets: 1 (Name it `xfusion-pub-subnet` with CIDR `10.0.1.0/24`)

Internet Gateway (IGW): Ensure an IGW is created and attached to `xfusion-pub-vpc`.

Click `Create VPC`.

### Enable Auto-Assign Public IP

***Subnet Settings***

In the VPC Console left menu, click Subnets.

Select `xfusion-pub-subnet`.

Click Actions > Edit subnet settings.

Check Enable auto-assign public IPv4 address.

Click Save.

### Configure Route to Internet Gateway

***Route Tables***

Go to Route tables and select the route table associated with `xfusion-pub-subnet`.

Under the Routes tab, click Edit routes.

Add route:

Destination: 0.0.0.0/0

Target: Internet Gateway (igw-xxxxxx)

Click Save changes.

### Create Security Group & Launch Instance

***EC2 Console***

Open the EC2 Console and click Launch instance.

Name: `xfusion-pub-ec2`.

AMI: Select any Ubuntu AMI.

Instance type: t2.micro.

Under Network settings, click Edit:

VPC: Select xfusion-pub-vpc.

Subnet: Select xfusion-pub-subnet.

Auto-assign public IP: Select Enable.

Under Inbound security group rules:

Type: SSH

Port: 22

Source: Anywhere-IPv4 (0.0.0.0/0)

Click Launch instance.

## Option 2: Using the AWS CLI

Execute the following script on your AWS client machine:

```Bash

# 1. Create Public VPC
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=xfusion-pub-vpc}]' \
  --query "Vpc.VpcId" \
  --output text)

# 2. Create Public Subnet
SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-pub-subnet}]' \
  --query "Subnet.SubnetId" \
  --output text)

# 3. Enable Auto-Assign Public IP on the Subnet
aws ec2 modify-subnet-attribute \
  --subnet-id $SUBNET_ID \
  --map-public-ip-on-launch

# 4. Create and Attach Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=xfusion-pub-igw}]' \
  --query "InternetGateway.InternetGatewayId" \
  --output text)

aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID

# 5. Create Route Table & Add Default Route to IGW
ROUTE_TABLE_ID=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=xfusion-pub-rt}]' \
  --query "RouteTable.RouteTableId" \
  --output text)

aws ec2 create-route \
  --route-table-id $ROUTE_TABLE_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

aws ec2 associate-route-table \
  --subnet-id $SUBNET_ID \
  --route-table-id $ROUTE_TABLE_ID

# 6. Create Security Group for SSH Access
SG_ID=$(aws ec2 create-security-group \
  --group-name xfusion-ssh-sg \
  --description "Allow SSH access on port 22" \
  --vpc-id $VPC_ID \
  --query "GroupId" \
  --output text)

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# 7. Retrieve Ubuntu AMI ID & Launch Instance
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --query "Images[0].ImageId" \
  --output text)

aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --subnet-id $SUBNET_ID \
  --security-group-ids $SG_ID \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-pub-ec2}]'
```

### Verification

Get the instance details:

```Bash


aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-pub-ec2" \
  --query "Reservations[0].Instances[0].[InstanceId, PublicIpAddress, State.Name]" \
  --output table
  ```
  
Test SSH connectivity to port 22:

```Bash


nc -zv <PUBLIC_IP> 22
# or
ssh -T root@<PUBLIC_IP>
```