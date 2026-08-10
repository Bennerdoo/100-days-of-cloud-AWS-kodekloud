# Question

As part of the migration, there were some components created under the AWS account. The Nautilus DevOps team created one EC2 instance where they forgot to enable the termination protection which is needed for this instance.
An instance named datacenter-ec2 already exists in us-east-1 region. 

> protection for the same.

# Step-by-step solution

## Option 1: Using the AWS Management Console

### 1. Select Region and Locate Instance:

***AWS Console.***

Log in to the AWS Management Console.
Check the top-right header and ensure the region is set to **US East (N. Virginia) us-east-1**.
Navigate to **EC2 > Instances**.
Select the check box next to **datacenter-ec2**.

### 2. Open Termination Protection Settings:

***Actions Menu.***

Click the **Actions** dropdown menu at the top.
Choose **Instance settings > Change termination protection**.


### 3. Enable Termination Protection:

***Configuration.***

Select the **Enable** checkbox under **Termination protection**.
Click **Save**.

## Option 2: Using the AWS CLI

Run these commands in your shell to retrieve the instance ID and enable termination protection:
```Bash
# 1. Retrieve the Instance ID for datacenter-ec2 in us-east-1
INSTANCE_ID=$(aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=datacenter-ec2" "Name=instance-state-name,Values=running,stopped" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text)

# 2. Enable termination protection
aws ec2 modify-instance-attribute \
  --region us-east-1 \
  --instance-id $INSTANCE_ID \
  --disable-api-termination
```

### Verification

**AWS Console**: Select datacenter-ec2, view the Details tab, and verify that Termination protection displays **Enabled**.

**AWS CLI**:Bash

```Bash
aws ec2 describe-instance-attribute \
  --region us-east-1 \
  --instance-id $INSTANCE_ID \
  --attribute disableApiTermination
  ```

  
The returned output will show **"Value": true**.