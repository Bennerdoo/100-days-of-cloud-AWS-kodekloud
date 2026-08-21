# Question

The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.
An IAM user named `iamuser_jim` and a policy named `iampolicy_jim` already exist. Attach the IAM policy `iampolicy_jim` to the IAM user `iamuser_jim`.

# Step by Step Solution

## Option 1: Using the AWS Management Console

### 1.Navigate to IAM Users:

**AWS Console.**

- Log in to the AWS Management Console.
- Search for IAM in the top search bar and select IAM under Services.In the left navigation pane, click Users.Select the user iamuser_jim.

### 2.Add Permissions:

**Permissions Tab.**

- On the user details page, ensure you are under the Permissions tab.Click the Add permissions dropdown button on the right and select Add permissions.

### 3.Attach Policy Directly:

**Policy Selection.**

- Select Attach policies directly.In the search box, search for iampolicy_jim.Check the box next to iampolicy_jim.Click Next at the bottom of the page.Review the policy details and click Add permissions.

## Option 2: Using the AWS CLI

Run the following AWS CLI script to look up the Policy ARN dynamically and attach it to iamuser_jim:
```Bash
# 1. Get the Policy ARN for iampolicy_jim
POLICY_ARN=$(aws iam list-policies \
  --scope Local \
  --query "Policies[?PolicyName=='iampolicy_jim'].Arn" \
  --output text)

# 2. Attach the policy to iamuser_jim
aws iam attach-user-policy \
  --user-name iamuser_jim \
  --policy-arn $POLICY_ARN
```

### Verification

To verify that the policy is successfully attached to the user:

- **AWS Console**: Check the Permissions tab under IAM > Users > iamuser_jim and verify iampolicy_jim is listed.

- **AWS CLI**: 
```Bash
aws iam list-attached-user-policies --user-name iamuser_jim
```
The returned output should display iampolicy_jim along with its ARN.