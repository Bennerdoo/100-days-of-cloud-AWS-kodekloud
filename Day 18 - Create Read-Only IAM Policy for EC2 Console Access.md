# Question

When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements.
- Create an IAM policy named `iampolicy_ammar` in `us-east-1` region, it must allow read-only access to the EC2 console, i.e this policy must allow users to view all instances, AMIs, and snapshots in the Amazon EC2 console.

# Step by Step Solution

## Option 1: Using the AWS Management Console

### 1.Navigate to IAM Policies:
**AWS Console.**
- Log in to the AWS Management Console.
- Search for IAM in the top search bar and select IAM under Services.
- In the left navigation pane, click Policies.
- Click the Create policy button.

### 2.Select AmazonEC2ReadOnlyAccess Managed Policy:Policy Editor
```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "ec2:Get*",
                "ec2:List*"
            ],
            "Resource": "*"
        }
    ]
}
```
- Click Next.

### 3.Set Policy Name and Create:Policy Details
```JSON
Policy Name: iampolicy_ammar
```

## Option 2: Using the AWS CLI
```Bash
aws iam create-policy \
  --policy-name iampolicy_ammar \
  --policy-document '{ 
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ec2:Describe*",
          "ec2:Get*",
          "ec2:List*"
        ],
        "Resource": "*"
      }
    ]
  }'
```

## Verification

To verify that the policy was successfully created:
```Bash
aws iam list-policies --query "Policies[?PolicyName=='iampolicy_ammar']" 
```