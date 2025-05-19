Here is a structured and clean documentation for setting up a **Frontend and Backend architecture using two Ubuntu EC2 servers**, connecting the backend to an RDS database, and preparing it for production.

---

# 🚀 Frontend & Backend EC2 Setup with RDS Integration and Route53

## 🧰 Objective

Deploy a full-stack web application on two separate **public Ubuntu EC2 instances**:

* One for **Frontend**
* One for **Backend**

Integrate the backend with **Amazon RDS** and set up a private **Route53 DNS** for internal hostname resolution.

---

## 📝 Step 0: Prerequisites

* A working **VPC** named `project-vpc`
* Two **public EC2 Ubuntu servers**

  * One for frontend
  * One for backend
* A configured **RDS MySQL** instance
* AWS CLI configured or access to the AWS console
* Internet access to the EC2 instances (via public IP or bastion)

---

## 🌐 Step 1: Frontend Setup (on Frontend EC2)

### 🛠️ 1.1 Create the Setup Script

SSH into your **frontend EC2 instance**, then run:

```bash
sudo vi /opt/frontend_setup.sh
```

Paste the following:

```bash
#!/bin/bash

sudo apt update -y
sudo apt install apache2 -y

curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs -y

sudo apt update -y
sudo npm install -g corepack -y

# Enable and activate yarn
corepack enable
corepack prepare yarn@stable --activate
```

Make it executable and run:

```bash
sudo chmod +x /opt/frontend_setup.sh
sudo /opt/frontend_setup.sh
```

---

### 🛠️ 1.2 Clone Git Repo and Configure

```bash
git clone https://github.com/CloudTechDevOps/2nd10WeeksofCloudOps-main.git
cd 2nd10WeeksofCloudOps-main/client
```

Edit the config file:

```bash
vi src/pages/config.js
```

Change:

```js
const API_BASE_URL = "http://publicip";
```

Example:

```js
const API_BASE_URL = "http://veera.nanri.co.in";
```

Use the **backend public IP or domain name**.

---

### 🛠️ 1.3 Build and Deploy Frontend

```bash
npm install
npm run build
sudo cp -r build/* /var/www/html
```

✅ Your frontend is now hosted using Apache at `http://<frontend-ec2-public-ip>`.

---

## 🖥️ Step 2: Backend Setup (on Backend EC2)

### 🔧 2.1 Create Backend Setup Script

SSH into your **backend EC2 instance**, then create:

```bash
sudo vi /opt/backend_setup.sh
```

Paste:

```bash
#!/bin/bash

sudo apt update -y

curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs -y

sudo apt update -y
sudo npm install -g corepack -y

corepack enable
corepack prepare yarn@stable --activate

sudo npm install -g pm2
```

Make it executable and run:

```bash
sudo chmod +x /opt/backend_setup.sh
sudo /opt/backend_setup.sh
```

---

### 🔧 2.2 Clone Git Repo and Configure .env

```bash
git clone https://github.com/CloudTechDevOps/2nd10WeeksofCloudOps-main.git
cd 2nd10WeeksofCloudOps-main/backend
```

Create the `.env` file:

```bash
vi .env
```

Add the following:

```
DB_HOST=book.rds.com
DB_USERNAME=admin
DB_PASSWORD="veera"
PORT=3306
```

Replace values according to your RDS configuration.

---

### 🔧 2.3 Install Dependencies and Start Backend

```bash
npm install
npm install dotenv
sudo pm2 start index.js --name "backendApi"
```

✅ Your backend API is now running and managed by PM2.

---

## 📦 Step 3: RDS and Route 53 Setup

### 📌 3.1 Create AMIs

Take **AMIs (Amazon Machine Images)** of both EC2 instances for future deployment via Terraform.

---

### 🌐 3.2 Setup Private Hosted Zone in Route53

1. Go to **Route53 > Hosted Zones > Create Hosted Zone**
2. Choose:

   * Name: `rds.com`
   * Type: **Private Hosted Zone**
   * VPC: Attach `project-vpc`

---

### 🌐 3.3 Create DNS Record for RDS

1. Within the `rds.com` hosted zone, create a **CNAME record**:

   * Name: `book`
   * Type: `CNAME`
   * Value: `<your-rds-endpoint>`

Example:

```
Record Name: book.rds.com
Record Type: CNAME
Value: mydb.cluster-abc123xyz456.us-east-1.rds.amazonaws.com
```

---

## 🛡️ Step 4: Access and Testing

* Use **book.rds.com** in your backend `.env` file to connect to RDS securely from within the VPC.
* Connect to the backend or frontend server via **bastion host** or public IP.
* Use `curl`, `ping`, or web browser to test connectivity.

---

## ✅ Summary

| Component     | Description                              |
| ------------- | ---------------------------------------- |
| Frontend EC2  | Apache + Node + React Static Build       |
| Backend EC2   | Node.js API + PM2 + `.env` for RDS       |
| RDS Host      | Defined via Route53 CNAME (book.rds.com) |
| DNS           | Private Hosted Zone attached to VPC      |
| Build Process | `npm install`, `npm run build`, `pm2`    |

---

Would you like this exported as `.md`, or want to automate EC2 setup and DNS via Terraform too?
