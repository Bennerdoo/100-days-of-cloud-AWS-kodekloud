# Question

During the migration process, several resources were created under the AWS account. Later on, some of these resources became obsolete as alternative solutions were implemented. Similarly, there is an instance that needs to be deleted as it is no longer in use.
- 1) Delete the ec2 instance named nautilus-ec2 present in us-east-1 region.
- 2) Before submitting your task, make sure instance is in terminated state.

# Step by Step Solution

## Option 1: Using the AWS Management Console1.
### Step 1- Select Region and Locate Instance:
**AWS Console.**
- Log in to the AWS Management Console.
- Ensure the top-right region selector is set to US East (N. Virginia) us-east-1.
- Navigate to EC2 > Instances.
- Select the check box next to nautilus-ec2.2.= 
### Disable Termination Protection (If Enabled):
**Pre-check.**
- If termination protection is enabled, click Actions > Instance settings > Change termination protection.
- Uncheck Enable and click Save.
### Terminate the Instance:
**Instance State.**
- With nautilus-ec2 selected, click the Instance state dropdown menu at the top.
- Click Terminate instance.
- Confirm by clicking Terminate in the pop-up modal.
### Wait for Terminated State:
**Status Verification.**
- Refresh the Instances list and monitor nautilus-ec2.
- Wait until its Instance state transitions from shutting-down to Terminated.

## Option 2: Using the AWS CLI
Run the following commands to disable protection, terminate the instance, and wait for it to reach the terminated state:
```Bash
# 1. Retrieve the Instance ID for nautilus-ec2
INSTANCE_ID=$(aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=nautilus-ec2" "Name=instance-state-name,Values=running,stopped" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text)

# 2. Disable termination protection (if enabled)
aws ec2 modify-instance-attribute \
  --region us-east-1 \
  --instance-id $INSTANCE_ID \
  --no-disable-api-termination

# 3. Terminate the instance
aws ec2 terminate-instances \
  --region us-east-1 \
  --instance-ids $INSTANCE_ID

# 4. Wait until the instance reaches 'terminated' state
aws ec2 wait instance-terminated \
  --region us-east-1 \
  --instance-ids $INSTANCE_ID
```

## Verification

To confirm termination:

**AWS Console**: Search for nautilus-ec2 in EC2 > Instances and verify its state is Terminated.

**AWS CLI**:
```Bash
aws ec2 describe-instances \
  --region us-east-1 \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].State.Name" \
  --output text
```
The returned output will be **terminated**.