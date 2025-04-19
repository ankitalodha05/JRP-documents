
# 🚀 AWS EC2 and Lambda CloudWatch Log Export to S3 – Detailed Setup Guide

This comprehensive guide walks you through the process of exporting logs from an EC2 instance to Amazon S3 via AWS CloudWatch and AWS Lambda. The setup involves four major components:
1. EC2 Log Collection via CloudWatch Agent
2. IAM Role Creation
3. S3 Bucket Configuration
4. Lambda Function for Export

---

## 📘 1. EC2 Instance Setup and CloudWatch Agent Configuration

### 🛠️ Step 1: Launch and Prepare EC2 Instance
- Sign in to the AWS Management Console.
- Navigate to **EC2 → Launch Instance**.
- Choose **Amazon Linux 2 AMI**.
- Assign an **IAM Role** with the `CloudWatchAgentServerPolicy` attached.
- Allow SSH access through the Security Group if needed.

### 🧰 Step 2: Install CloudWatch Agent
SSH into your EC2 instance and run the following commands:

```bash
sudo yum install amazon-cloudwatch-agent -y
```

Create the configuration file:

```bash
sudo vi /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Paste this configuration to collect `/var/log/*`:

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

### ⚙️ Step 3: Start and Enable CloudWatch Agent

Run the following command to apply the configuration and start the agent:

```bash
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

Then enable the service to start on boot:

```bash
sudo systemctl start amazon-cloudwatch-agent
sudo systemctl enable amazon-cloudwatch-agent
sudo systemctl status amazon-cloudwatch-agent
```

---

## 🔐 2. IAM Role Configuration for Lambda

### 🎯 Permissions Needed
Create an IAM Role for your Lambda function with the following AWS managed policies:
- `AmazonS3FullAccess`
- `AWSLambdaExecute`
- `CloudWatchFullAccess`

### 🔄 Trust Relationship

This policy defines what services can assume the role. Paste this JSON into the **Trust Relationships** tab:

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

## 🪣 3. S3 Bucket Setup for Log Storage

### 🧱 Step 1: Create a Bucket
- Go to **S3 → Create bucket**
- Name it `ashrafbucket345`
- Keep all defaults or set your required configuration

### 🔐 Step 2: Add Bucket Policy

Go to **Permissions → Bucket Policy** and paste:

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

## 🧠 4. Lambda Function to Export Logs

### 📝 Purpose
This function exports logs from the specified CloudWatch log group to the S3 bucket, based on a date range.

### 🧑‍💻 Python Code

Create a Lambda function with the below code:

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

### 🕒 Schedule with EventBridge
- Go to **Amazon EventBridge**
- Create a rule to run the Lambda function **once per day**
- This ensures logs are exported daily

---

## ✅ Conclusion

Once the above setup is complete:
- Your EC2 instance logs will be collected and stored in CloudWatch.
- The Lambda function will extract and save them to your S3 bucket daily.
- This setup can be used for backup, compliance, and analytics.

