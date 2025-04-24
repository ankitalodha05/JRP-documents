**Beginner-Friendly Guide: RDS + Redis Caching Setup on AWS**

---

### 🔖 Overview

This document outlines the step-by-step process to:

- Set up an EC2 instance with Amazon Linux.
- Create and connect to a MySQL RDS instance.
- Configure a Redis ElastiCache cluster.
- Write and run a Python script that caches data from RDS to Redis.

---

### 🔢 Step 1: Launch EC2 Instance

1. Go to **AWS EC2 Console**.
2. Click **Launch Instance**.
3. Select **Amazon Linux 2 AMI**.
4. Choose **t2.micro** (Free tier eligible).
5. Configure:
   - Network: Same VPC where you plan to create RDS and ElastiCache.
   - Key pair: Create or select a key pair for SSH access.
   - Security Group: Allow SSH (port 22).
6. Launch the instance.
7. Connect using SSH:
   ```bash
   ssh -i /path/to/key.pem ec2-user@<EC2-Public-IP>
   ```

---

### 📁 Step 2: Create RDS MySQL Database

1. Go to **AWS RDS Console**.
2. Click **Create Database**.
3. Choose:
   - Engine: **MySQL**
   - Template: Free tier
   - DB Instance Identifier: `db-instance`
   - Master Username: `admin`
   - Password: `admin123`
4. Enable **Public access** for testing (disable in prod).
5. Launch DB and **note the endpoint**.

---

### 🛠️ Step 3: Create Redis ElastiCache Cluster

1. Go to **AWS ElastiCache Console**.
2. Choose **Redis** and click **Create Cluster**.
3. Configure:
   - Name: `ankitacache`
   - Node type: `cache.t2.micro`
   - Subnet group: Default or matching your EC2
   - Security group: Same as EC2 or one that allows traffic from EC2 on port 6379
4. Click **Create**.
5. After creation, copy the **endpoint** (exclude port).

---

### 📆 Step 4: Install Dependencies on EC2

SSH into your EC2 instance and run:

```bash
sudo yum install mariadb105-server -y
sudo yum install python3-pip -y
pip3 install pymysql redis
```

---

### 🛡️ Step 5: Connect to RDS from EC2

```bash
mysql -h <RDS-ENDPOINT> -u admin -p
```

Then execute the following SQL:

```sql
CREATE DATABASE test;
USE test;

CREATE TABLE users (
    user_id VARCHAR(50) PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    email VARCHAR(100)
);

INSERT INTO users (user_id, first_name, last_name, email)
VALUES ('12345', 'John', 'Doe', 'john.doe@example.com');
```

---

### 📄 Step 6: Create the Python Script (`cache.py`)

Create a file using:

```bash
nano cache.py
```

Paste the following code:

```python
import pymysql
import redis
import json
import sys

# Redis config
redis_client = redis.Redis(
    host='ankitacache-i1jl16.serverless.use1.cache.amazonaws.com',
    port=6379,
    ssl=True,
    decode_responses=True,
    socket_timeout=5
)

# RDS config
RDS_HOST = 'db-instance.cn2g4mowubxp.us-east-1.rds.amazonaws.com'
RDS_USER = 'admin'
RDS_PASSWORD = 'admin123'
RDS_DB_NAME = 'test'
TABLE_NAME = 'users'

def fetch_data_from_rds():
    try:
        connection = pymysql.connect(
            host=RDS_HOST,
            user=RDS_USER,
            password=RDS_PASSWORD,
            database=RDS_DB_NAME
        )
        print("🔗 Connected to RDS")
        with connection.cursor() as cursor:
            cursor.execute(f"SELECT * FROM {TABLE_NAME} LIMIT 10;")
            rows = cursor.fetchall()
            return rows
    except Exception as e:
        print("❌ RDS Error:", e)
        return None
    finally:
        if 'connection' in locals():
            connection.close()

def main():
    cache_key = 'cached_table_data'
    bypass_cache = "--refresh" in sys.argv

    if not bypass_cache:
        cached_data = redis_client.get(cache_key)
        if cached_data:
            print("✅ Fetched from Redis cache:")
            print(json.loads(cached_data))
            return

    print("⚙️ No cache or refresh requested. Fetching from RDS...")
    data = fetch_data_from_rds()
    if data:
        redis_client.set(cache_key, json.dumps(data), ex=90)
        print("📦 Cached in Redis:")
        print(data)
    else:
        print("⚠️ No data fetched.")

if __name__ == "__main__":
    main()
```

---

### 🚀 Step 7: Run the Script

```bash
python3 cache.py
```

- First run: Fetches from **RDS** and caches in Redis.
- Next run (within 90 seconds): Fetches from **Redis**.
- Use `--refresh` to bypass the cache:

```bash
python3 cache.py --refresh
```

---

### 📊 Summary

- Redis TTL (Time To Live) is set to 90 seconds.
- If data is updated in RDS within TTL, Redis won’t reflect changes immediately.
- After TTL expires, Redis pulls fresh data from RDS.

---

