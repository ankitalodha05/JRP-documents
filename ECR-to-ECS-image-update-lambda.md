Here's a well-formatted document based on your input for **automatically updating an Amazon ECS Fargate service with a new ECR image using AWS Lambda**. You can use this for internal documentation, blogs, or project wikis.

---

# 🐳 ECR to ECS Image Update using AWS Lambda

## ✅ Goal

Automatically update your **ECS Fargate service** when a **new Docker image** is pushed to **Amazon ECR**, by:

1. Detecting the image push via **EventBridge**
2. Registering a **new ECS task definition**
3. Updating the **ECS service** with the new task

---

## ✅ Prerequisites

Before you begin, ensure the following are in place:

1. ✅ You have a **working ECS Fargate cluster and service**.
2. ✅ Your image is pushed to **Amazon ECR** (not Docker Hub).
3. ✅ Your ECS task has a valid **IAM task execution role** with ECR pull permissions.

---

## 🔹 1. Push Docker Image to ECR

```bash
#!/bin/bash

# Create ECR repo (if not already created)
aws ecr create-repository --repository-name hotstar

# Login to ECR
aws ecr get-login-password | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com

# Build Docker image
docker build -t hotstar .

# Tag the image
docker tag hotstar:latest <account-id>.dkr.ecr.<region>.amazonaws.com/hotstar:latest

# Push to ECR
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/hotstar:latest
```

---

## 🔹 2. Create the Lambda Function

### Configuration

* **Runtime**: Python 3.9 or 3.11
* **Timeout**: 30 seconds
* **Permissions**: Attach a role with:

  * `AmazonECSFullAccess`
  * `AmazonEC2ContainerRegistryReadOnly`
  * `AWSLambdaBasicExecutionRole`

### Environment Variables

| Key              | Value                      |
| ---------------- | -------------------------- |
| `ECS_CLUSTER`    | your-cluster-name          |
| `ECS_SERVICE`    | your-service-name          |
| `CONTAINER_NAME` | container name in task def |

---

## 🔹 3. Lambda Code

```python
import boto3
import os
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

ecs_client = boto3.client('ecs')

def lambda_handler(event, context):
    try:
        cluster_name = os.environ['ECS_CLUSTER']
        service_name = os.environ['ECS_SERVICE']
        container_name = os.environ['CONTAINER_NAME']

        # Get image details from the event
        repository_name = event.get('detail', {}).get('repository-name')
        image_tag = event.get('detail', {}).get('image-tag')
        if not repository_name or not image_tag:
            raise ValueError("Missing 'repository-name' or 'image-tag' in event['detail']")

        account_id = boto3.client('sts').get_caller_identity()['Account']
        region = os.environ.get("AWS_REGION")
        image_uri = f"{account_id}.dkr.ecr.{region}.amazonaws.com/{repository_name}:{image_tag}"
        logger.info(f"Updating image to: {image_uri}")

        # Describe current task definition
        response = ecs_client.describe_services(cluster=cluster_name, services=[service_name])
        task_def_arn = response['services'][0]['taskDefinition']
        task_def = ecs_client.describe_task_definition(taskDefinition=task_def_arn)

        # Modify container definition with new image
        container_definitions = task_def['taskDefinition']['containerDefinitions']
        updated = False
        for container in container_definitions:
            if container['name'] == container_name:
                container['image'] = image_uri
                updated = True

        if not updated:
            raise ValueError(f"Container '{container_name}' not found in task definition.")

        # Register new task definition
        new_task_def = ecs_client.register_task_definition(
            family=task_def['taskDefinition']['family'],
            executionRoleArn=task_def['taskDefinition']['executionRoleArn'],
            networkMode=task_def['taskDefinition']['networkMode'],
            containerDefinitions=container_definitions,
            requiresCompatibilities=task_def['taskDefinition']['requiresCompatibilities'],
            cpu=task_def['taskDefinition']['cpu'],
            memory=task_def['taskDefinition']['memory'],
        )

        # Update ECS service
        ecs_client.update_service(
            cluster=cluster_name,
            service=service_name,
            taskDefinition=new_task_def['taskDefinition']['taskDefinitionArn']
        )

        logger.info("✅ ECS service updated successfully.")
        return {"message": "ECS service updated with new image"}

    except Exception as e:
        logger.error(f"❌ Error updating ECS service: {str(e)}")
        return {"error": str(e)}
```

---

## 🔹 4. Create EventBridge Rule (Trigger)

### Steps:

1. Go to **Amazon EventBridge Console**
2. Click **Create Rule**
3. Name it: `ECRImagePushTrigger`
4. Event source: **AWS events**
5. **Event pattern**:

```json
{
  "source": ["aws.ecr"],
  "detail-type": ["ECR Image Action"],
  "detail": {
    "action-type": ["PUSH"]
  }
}
```

6. **Target**: Your Lambda function
7. Click **Create**

---

## 🔹 5. Test End-to-End

### 🔁 Push a New Image

```bash
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/hotstar:latest
```

This will trigger:

* ✅ The Lambda function
* ✅ ECS Task Definition Update
* ✅ ECS Service Deployment

### ✅ Verify

Go to the **ECS Console** → check if the running task is using the new image.

---

## ⚠️ Common Error

### ❗ Error Message:

```json
{ "error": "Missing 'repository-name' or 'image-tag' in event['detail']" }
```

### 🛠 Solution:

Update your **Lambda test event** to:

```json
{
  "detail": {
    "repository-name": "hotstar",
    "image-tag": "latest"
  }
}
```

---

Let me know if you'd like this exported as a **PDF**, **Markdown**, or added to a **GitHub repository**.
