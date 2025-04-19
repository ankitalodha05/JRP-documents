
# AWS EC2 and Lambda Log Export Setup Guide

This document provides a step-by-step guide to set up EC2 CloudWatch logs export to S3 using a Lambda function.

---

## 1. EC2 Instance Configuration

### Step 1: Launch EC2 Instance

- Go to **AWS Console → EC2 → Launch Instance**
- Choose your preferred AMI (e.g., Amazon Linux)
- Attach an **IAM Role** with `CloudWatchAgentServerPolicy` permission

---

### Step 2: Install and Configure CloudWatch Agent

Run the following on your EC2 instance:

```bash
sudo yum install amazon-cloudwatch-agent -y
sudo vi /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Paste the configuration below:

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/*",
            "log_group_name": "LOG-FROM-EC2",
            "log_stream_name": "dev_1",
            "retention_in_days": 1
          }
        ]
      }
    }
  }
}
```

---

### Step 3: Start the CloudWatch Agent

Run:

```bash
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s

sudo systemctl start amazon-cloudwatch-agent
sudo systemctl enable amazon-cloudwatch-agent
sudo systemctl status amazon-cloudwatch-agent
```

---

## 2. IAM Role for Lambda

### Required Policies

- `AmazonS3FullAccess`
- `AWSLambdaExecute`
- `CloudWatchFullAccess`

### Trust Relationship JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "textract.amazonaws.com",
          "glue.amazonaws.com",
          "delivery.logs.amazonaws.com",
          "transcribe.amazonaws.com",
          "rds.amazonaws.com",
          "logs.amazonaws.com",
          "dms.amazonaws.com",
          "lakeformation.amazonaws.com",
          "events.amazonaws.com",
          "lambda.amazonaws.com",
          "datapipeline.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

## 3. S3 Bucket Configuration

### Create Bucket

- Name: `ashrafbucket345`

### Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "uploadlogs",
      "Effect": "Allow",
      "Principal": {
        "Service": "logs.us-east-1.amazonaws.com"
      },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::ashrafbucket345"
    },
    {
      "Sid": "uploadlogs",
      "Effect": "Allow",
      "Principal": {
        "Service": "logs.us-east-1.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::ashrafbucket345/*"
    }
  ]
}
```

---

## 4. Lambda Function to Export Logs to S3

### Python Code

```python
import boto3
import os
import datetime

GROUP_NAME = "aws/lambda/applicationlambda"
DESTINATION_BUCKET = "ashrafbucket345"
PREFIX = "CloudQuickLabs"
NDAYS = 1

nDays = int(NDAYS)
currentTime = datetime.datetime.now()
StartDate = currentTime - datetime.timedelta(days=nDays)
EndDate = currentTime - datetime.timedelta(days=nDays - 1)

fromDate = int(StartDate.timestamp() * 1000)
toDate = int(EndDate.timestamp() * 1000)
BUCKET_PREFIX = os.path.join(PREFIX, StartDate.strftime('%Y{0}%m{0}%d').format(os.path.sep))

def lambda_handler(event, context):
    client = boto3.client('logs')
    response = client.create_export_task(
        logGroupName=GROUP_NAME,
        fromTime=fromDate,
        to=toDate,
        destination=DESTINATION_BUCKET,
        destinationPrefix=BUCKET_PREFIX
    )
    print(response)
```

> 💡 Schedule this Lambda with **Amazon EventBridge** for daily log exports.
