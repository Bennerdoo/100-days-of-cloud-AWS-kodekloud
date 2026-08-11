# Question

The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.
There is an instance named devops-ec2 and an elastic-ip named devops-ec2-eip in us-east-1 region. Attach the devops-ec2-eip elastic-ip to the devops-ec2 instance.

# Step-by-Step Solution

## Option 1: Using the AWS Management Console

### 1. Allocate a New Elastic IP (If not already allocated)
If you don't have an Elastic IP ready, allocate one:
- Go to **EC2 Dashboard > Network & Security > Elastic IPs**.
- Click **Allocate Elastic IP address**.
- Ensure **Amazon's pool of IPv4 addresses** is selected.
- Click **Allocate**.
- Note the **Elastic IP address**.

> **Note**: In this lab, the Elastic IP `devops-ec2-eip` is already allocated. You can skip this step and just note its IP address.

### 2. Locate and Stop the Target Instance
- Go to **EC2 Dashboard > Instances**.
- Select the instance **devops-ec2**.
- Click **Instance state > Stop instance** and confirm.
- Wait for the instance state to become **Stopped**.

### 3. Disassociate Existing Elastic IP (If any)
- Check if **devops-ec2** has an existing Elastic IP attached.
  - If yes, select the instance.
  - Click **Actions > Network & Security > Change security groups** (just kidding, wrong one).
  - Click **Actions > Network & Security > Manage IP addresses**.
  - Under **Elastic IP addresses**, click **Manage** (or similar action to disassociate).
  - Select the existing EIP and click **Disassociate IP address**.

### 4. Associate the Elastic IP with the Instance
- Select the **devops-ec2** instance.
- Click **Actions > Network & Security > Manage IP addresses**.
- Click **Associate IP address**.
- Select the Elastic IP **devops-ec2-eip** from the dropdown.
- Ensure the correct network interface is selected (usually `eth0`).
- Click **Associate IP address**.

### 5. Start the Instance
- Select **devops-ec2**.
- Click **Instance state > Start instance**.
- Wait for the instance state to become **Running**.
- Verify the **Public IPv4 address** now shows the Elastic IP.

## Option 2: Using the AWS CLI

Replace `[Your-Elastic-IP]` with the actual IP address of `devops-ec2-eip`.

```Bash
# 1. Get the Instance ID for devops-ec2
INSTANCE_ID=$(aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=devops-ec2" "Name=instance-state-name,Values=running,stopped" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text)

# 2. Stop the instance if running
aws ec2 stop-instances --instance-ids $INSTANCE_ID
aws ec2 wait instance-stopped --instance-ids $INSTANCE_ID

# 3. Disassociate the existing EIP (if any)
# First, get the Allocation ID of the EIP
ALLOCATION_ID=$(aws ec2 describe-addresses \
  --region us-east-1 \
  --filters "Name=public-ip,Values=[Your-Elastic-IP]" \
  --query "Addresses[0].AllocationId" \
  --output text)

# Disassociate it
if [ "$ALLOCATION_ID" != "None" ]; then
  aws ec2 disassociate-address \
    --region us-east-1 \
    --association-id $ALLOCATION_ID
fi

# 4. Associate the EIP with the instance
aws ec2 associate-address \
  --region us-east-1 \
  --instance-id $INSTANCE_ID \
  --allocation-id $ALLOCATION_ID

# 5. Start the instance
aws ec2 start-instances --instance-ids $INSTANCE_ID
```

## Verification

Check the EC2 Console or use the AWS CLI to confirm:
```Bash
aws ec2 describe-instances \
  --region us-east-1 \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].PublicIpAddress" \
  --output text
```
The output should be the Elastic IP address `devops-ec2-eip`.