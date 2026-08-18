# Question

When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements:

For this task, create an IAM user named `iamuser_james`.

# Step-by-Step Solution

## Option 1: Using the AWS Management Console

### 1. Navigate to IAM Dashboard:

***AWS Console.***

Log in to the AWS Management Console.
Search for IAM in the top search bar and select IAM under Services.
### 2. Open User Creation Form:

***Users Section.***

In the left navigation pane, click Users.
Click the Create user button at the top right.
### 3. Configure User Name:

***User Details.***

In the User name field, enter iamuser_james.
Leave console access unchecked (unless management console access is specifically required).
Click Next.
### 4. Finalize User Creation:

***Permissions & Review.***

On the Set permissions step, configure permissions/groups as required (or click Next to skip).
Review the details on the final step and click Create user.


## Option 2: Using the AWS CLI

Run the following command in your terminal to create the IAM user directly:

***Bash***

```Bash
aws iam create-user --user-name iamuser_james
```

## Verification

To verify that the IAM user has been created:

***AWS Console:***

Navigate to IAM > Users and verify iamuser_james is listed.

***AWS CLI:***

```Bash
aws iam get-user --user-name iamuser_james
```

The returned JSON output will display the details and ARN for iamuser_james.