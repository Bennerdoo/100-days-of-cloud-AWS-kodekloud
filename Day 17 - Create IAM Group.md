# Question

The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.
Create an IAM group named `iamgroup_james`.

# Step by step solution

## Option 1: Using the AWS Management Console

### 1. Navigate to IAM Dashboard:

***AWS Console.***

- Log in to the AWS Management Console.
- Search for IAM in the top search bar and select IAM under Services.

### 2. Open User Group Creation Form:

***User Groups.***

- In the left navigation pane, click **User groups**.
- Click the **Create group** button at the top right.

### 3. Configure Group Name and Finalize:

***Group Details.***

- In the **User group name** field, enter `iamgroup_james`.
- Attach any required policies or add users if needed (or leave blank).
- Scroll to the bottom and click **Create group**.


## Option 2: Using the AWS CLI

Run the following command in your terminal to create the IAM group directly:

```Bash
aws iam create-group --group-name iamgroup_james
```

### Verification

To verify that the IAM group has been created:

- **AWS Console**: Navigate to **IAM > User groups** and check that `iamgroup_james` is listed.

- **AWS CLI**:

```Bash
aws iam get-group --group-name iamgroup_james
```


The returned JSON output will display the details and ARN for `iamgroup_james`.