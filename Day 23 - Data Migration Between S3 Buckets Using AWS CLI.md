# Question

As part of a data migration project, the team lead has tasked the team with migrating data from an existing S3 bucket to a new S3 bucket. The existing bucket contains a substantial amount of data that must be accurately transferred to the new bucket. The team is responsible for creating the new S3 bucket and ensuring that all data from the existing bucket is copied or synced to the new bucket completely and accurately. It is imperative to perform thorough verification steps to confirm that all data has been successfully transferred to the new bucket without any loss or corruption.

As a member of the Nautilus DevOps Team, your task is to perform the following:

- Create a New Private S3 Bucket: Name the bucket xfusion-sync-18678.
- Data Migration: Migrate the entire data from the existing xfusion-s3-3812 bucket to the new xfusion-sync-18678 bucket.
- Ensure Data Consistency: Ensure that both buckets have the same data.
- Use AWS CLI: Use the AWS CLI to perform the creation and data migration tasks.

Notes:
- Create the resources only in us-east-1 region.
- To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:

# Step By Step Solution

1.Create the Bucket:AWS CLI
Use aws s3api (or aws s3 mb)

```Bash
aws s3api create-bucket --bucket xfusion-sync-18678 --region us-east-1
```

Or using s3 mb:

```Bash
aws s3 mb s3://xfusion-sync-18678 --region us-east-1
```

2.Sync the Data:

**AWS CLI**
Migrate all content from the source bucket to the new bucket

```Bash
aws s3 sync s3://xfusion-s3-3812 s3://xfusion-sync-18678 --region us-east-1
```

3.Verify Data Consistency:

**AWS CLI**
Check object counts in both buckets

```Bash
aws s3 ls s3://xfusion-s3-3812 --recursive --summarize
aws s3 ls s3://xfusion-sync-18678 --recursive --summarize
```

Confirm no differences remain:

```Bash
aws s3 sync s3://xfusion-s3-3812 s3://xfusion-sync-18678 --dryrun
```