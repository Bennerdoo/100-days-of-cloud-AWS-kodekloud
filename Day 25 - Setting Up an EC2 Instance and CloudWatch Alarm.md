# Question

The Nautilus DevOps team has been tasked with setting up an EC2 instance for their application. To ensure the application performs optimally, they also need to create a CloudWatch alarm to monitor the instance's CPU utilization. The alarm should trigger if the CPU utilization exceeds 90% for one consecutive 5-minute period. To send notifications, use the SNS topic named `xfusion-sns-topic` which is already created.

Launch EC2 Instance: Create an EC2 instance named `xfusion-ec2` using any appropriate Ubuntu AMI.

Create CloudWatch Alarm: Create a CloudWatch alarm named `xfusion-alarm` with the following specifications:

- Statistic: Average
- Metric: CPU Utilization
- Threshold: >= 90% for 1 consecutive 5-minute period.
- Alarm Actions: Send a notification to `xfusion-sns-topic`.

# Step-by-step Solution

## **Option 1: Using the AWS Management Console**

### 1. Launch the EC2 Instance:

**EC2 Console:**
Open the EC2 Console and click Launch instance. Set Name to xfusion-ec2. Under Application and OS Images, select an Ubuntu AMI. Choose your Instance type (e.g., t2.micro), configure key pair/network settings, and click Launch instance.

### 2. Create the CloudWatch Alarm:

**CloudWatch Console:**
Open the CloudWatch Console. In the left navigation pane, expand Alarms and click All alarms. Click Create alarm. Click Select metric, search for EC2 > By Instance ID, and select CPUUtilization for xfusion-ec2 (i-...). Click Select metric.

### 3. Configure Threshold Specifications:

**Metric and conditions:**
Set Statistic to Average. Set Period to 5 minutes. Under Conditions: Set Threshold type to Static. Set Whenever CPUUtilization is... to Greater than/Equal to (>=). Set than the threshold to 90. Click Next.

### 4. Configure Alarm Actions & SNS:

**Configure actions:**
Under Alarm state trigger, select In alarm. Under Notification, select Select an existing SNS topic. Choose xfusion-sns-topic from the dropdown menu. Click Next.

### 5. Name and Create Alarm:

**Name and review:**
Enter xfusion-alarm as the Alarm name. Click Next, review your settings, and click Create alarm.

## **Option 2: Using the AWS CLI**

Run the following commands in your terminal to provision the EC2 instance, retrieve its ID, find the SNS topic ARN, and create the CloudWatch alarm:

```Bash
# 1. Get an Ubuntu AMI ID
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --query "Images[0].ImageId" \
  --output text)

#### 2. Launch xfusion-ec2 instance
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
  --query "Instances[0].InstanceId" \
  --output text)

#### 3. Get the ARN for the existing xfusion-sns-topic
SNS_ARN=$(aws sns list-topics \
  --query "Topics[?contains(TopicArn, 'xfusion-sns-topic')].TopicArn" \
  --output text)

#### 4. Create the CloudWatch alarm
aws cloudwatch put-metric-alarm \
  --alarm-name xfusion-alarm \
  --alarm-description "Alarm when CPU exceeds 90% for 5 minutes" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 90 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --dimensions Name=InstanceId,Value=$INSTANCE_ID \
  --alarm-actions $SNS_ARN
```

### Verification

**AWS Console**

Navigate to CloudWatch > Alarms and confirm that xfusion-alarm is in the OK state.

**AWS CLI**

```Bash
aws cloudwatch describe-alarms --alarm-names xfusion-alarm
```