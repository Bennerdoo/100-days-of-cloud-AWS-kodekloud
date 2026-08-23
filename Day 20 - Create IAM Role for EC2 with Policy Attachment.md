# Question

When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements:
Create an IAM role as below:
1) IAM role name must be iamrole_kareem.
2) Entity type must be AWS Service and use case must be EC2.
3) Attach a policy named iampolicy_kareem.

# Step by Step Solution

## Option 1: Using the AWS Management Console

### 1. Navigate to IAM Roles:
AWS Console.
- Log in to the AWS Management Console.
- Search for IAM in the top search bar and select IAM under Services.
- In the left navigation pane, click Roles.
- Click the Create role button.

### 2. Select Entity Type and Use Case:
Trusted Entity.
- Under Trusted entity type, select AWS service.
- Under Use case, select EC2.
- Click Next.

### 3. Attach the Policy:
Permissions.
- In the policy search box, type iampolicy_kareem.
- Select the checkbox next to iampolicy_kareem.
- Click Next.

### 4. Finalize Role Creation:
Name and Review.
- In the Role name field, enter iamrole_kareem.
- Review the trust policy and permissions.
- Scroll to the bottom and click Create role.

## Option 2: Using the AWS CLI

Run the following commands to construct the trust policy, create the role, and attach the policy:
```Bash

# 1. Get the ARN for iampolicy_kareem
POLICY_ARN=$(aws iam list-policies \
  --scope Local \
  --query "Policies[?PolicyName=='iampolicy_kareem'].Arn" \
  --output text)

# 2. Create the IAM Role with EC2 trust policy
aws iam create-role \
  --role-name iamrole_kareem \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": "ec2.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }'

# 3. Attach iampolicy_kareem to the role
aws iam attach-role-policy \
  --role-name iamrole_kareem \
  --policy-arn $POLICY_ARN
```

# Verification

To verify that the role has been created and configured correctly:

- **AWS Console**: Navigate to IAM > Roles > iamrole_kareem. Check the Trust relationships tab (should list ec2.amazonaws.com) and the Permissions tab (should list iampolicy_kareem).

- **AWS CLI**:

```Bash
aws iam list-attached-role-policies --role-name iamrole_kareem
```