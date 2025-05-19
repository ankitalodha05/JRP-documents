Here's a clear and professional document for automating Jenkins backup to an S3 bucket using a scheduled cron job.

---

# 📦 Jenkins Backup to Amazon S3 – Step-by-Step Guide

This guide helps you create a **daily automated backup of Jenkins data** from an EC2 instance to an **Amazon S3 bucket** using a shell script and cron.

---

## ✅ Prerequisites

* Jenkins is installed and running on an EC2 instance.
* AWS CLI is configured or the EC2 instance has an appropriate IAM role attached.
* An S3 bucket is already created for storing backups.

---

## 🧾 Step 1: Create Jenkins Backup Script

### 🔹 SSH into the Jenkins EC2 Instance

```bash
ssh -i <your-key.pem> ec2-user@<jenkins-ec2-public-ip>
```

### 🔹 Create the Backup Script File

```bash
sudo vi /opt/jenkins_backup_to_s3.sh
```

### 🔹 Paste the Script Below into the File

```bash
#!/bin/bash

# Variables
BACKUP_DIR="/var/lib/jenkins"
BACKUP_FILE="/tmp/jenkins-backup-$(date +%F).tar.gz"
S3_BUCKET="s3://your-s3-bucket-name/backups"

# Create tar.gz archive of Jenkins data
tar -czf "$BACKUP_FILE" "$BACKUP_DIR"

# Upload the archive to S3
aws s3 cp "$BACKUP_FILE" "$S3_BUCKET"

# Optionally remove local backup after upload
rm -f "$BACKUP_FILE"
```

> 🛠️ Replace `your-s3-bucket-name` with your actual bucket name.

---

## 🔐 Step 2: Make the Script Executable

```bash
sudo chmod +x /opt/jenkins_backup_to_s3.sh
```

---

## 🔐 Step 3: Ensure AWS S3 Access

### 🔹 Option 1: IAM Role (Recommended)

Ensure your EC2 instance has an IAM role attached with the following policy:

```json
{
  "Effect": "Allow",
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::your-s3-bucket-name/*"
}
```

### 🔹 Option 2: AWS CLI Credentials

Alternatively, configure the AWS CLI:

```bash
aws configure
```

---

## ⏰ Step 4: Schedule Daily Backup with Cron

### 🔹 Open Crontab

```bash
crontab -e
```

### 🔹 Add the Following Line

```bash
0 2 * * * /opt/jenkins_backup_to_s3.sh >> /var/log/jenkins_backup.log 2>&1
```

* This will run the backup **daily at 2:00 AM**.
* Logs will be written to `/var/log/jenkins_backup.log`.

---

## 🧪 Optional: Test the Script Manually

```bash
sudo /opt/jenkins_backup_to_s3.sh
```

Check the S3 bucket to verify the `.tar.gz` backup file is uploaded.

---

## 📌 Summary

| Task                  | Description                                |
| --------------------- | ------------------------------------------ |
| Script Location       | `/opt/jenkins_backup_to_s3.sh`             |
| Backup Directory      | `/var/lib/jenkins`                         |
| S3 Bucket             | Replace with your actual bucket name       |
| Backup Schedule       | Daily at 2 AM (via cron)                   |
| IAM Permission Needed | `s3:PutObject` to your backup folder in S3 |

---

Would you like this as a `.md` file or want me to add email alerting, versioning, or retention policies?
