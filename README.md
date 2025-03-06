# 🔥 **Step 1: Set up the MySQL RDS Instance**

## 1.1 - Create an AWS Account (if not already done)
- Go to [https://aws.amazon.com](https://aws.amazon.com) and sign up for a **free-tier account** if you don't have one.

## 1.2 - Go to RDS Console
- Navigate to [https://console.aws.amazon.com/rds](https://console.aws.amazon.com/rds).

## 1.3 - Create MySQL Database
1. Log in to the AWS Management Console
2. Navigate to RDS service
3. Click "Create database"
4. Choose the following settings:
   - Select "Standard create"
   - Engine type: MySQL
   - Version: MySQL 8.0.28 (or latest available in free tier)
   - Templates: Free tier
   - DB instance identifier: employee-db (or your preferred name)
   - Master username: admin (or your preferred username)
   - Master password: (create a secure password)
   - DB instance class: db.t2.micro or db.t3.micro (free tier eligible)
   - Storage: General Purpose SSD, 20 GB (free tier limit)
   - Enable storage autoscaling: No (to avoid unexpected charges)
   - VPC: Default VPC
   - Public access: Yes (for this project only, normally not recommended)
   - VPC security group: Create new
   - Availability Zone: No preference
   - Database authentication: Password authentication
   - Initial database name: employees
   - Backup retention period: 1 day (minimum)
   - Disable automated backups: No
   - Encryption: Disable encryption (for free tier)
   - Monitoring: Disable enhanced monitoring (for free tier)
   - Maintenance: Enable auto minor version upgrade

5. Click "Create database"

## 1.4 - Security Group Configuration
1. Go to EC2 > Security Groups
2. Find the security group associated with your RDS instance
3. Edit inbound rules
4. Add a rule:
   - Type: MySQL/Aurora
   - Protocol: TCP
   - Port Range: 3306
   - Source: 0.0.0.0/0 (for testing only - in production you would restrict this)
5. Save rules

## 1.5 - Finalize Creation
- Review and Launch.
- It may take 5-10 minutes for the database to become available.

---

# ✅ **Step 2: Load Sample Data into MySQL RDS**

---

## 📌 2.1 - Clone the Sample Database Repo

This repo contains the **employees.sql** file, which has the dataset you need to load into RDS.

### Commands:
```bash
git clone https://github.com/datacharmer/test_db
cd test_db
```

### Explanation:
- `test_db` repo has the **employees.sql** file which contains all the SQL commands needed to create tables, insert data, etc.
- This file has **300,024 employee records** (as required).

---

## 📌 2.2 - Install MySQL Client (MySQL Shell)

### Why?
You need a MySQL client to connect to your RDS instance and run the SQL script.

### Install Links:
- [MySQL Shell Download Page](https://dev.mysql.com/downloads/shell/)
- Choose the version matching your OS (Windows, Mac, Linux).

### Verify Installation:
```bash
mysqlsh --version
```
You should see something like:
```
MySQL Shell 8.4.x
```

---

## 📌 2.3 - Connect to Your RDS Instance

### 1. Go to AWS RDS Console
- Open: [https://console.aws.amazon.com/rds](https://console.aws.amazon.com/rds)
- Find your **RDS Endpoint** under:
    - RDS > Databases > Your Database > **Connectivity & Security tab**

### 2. Use MySQL Client to Connect

Example Command:
```bash
mysqlsh -h your-rds-endpoint.rds.amazonaws.com -u admin -p
```
- Replace:
    - `your-rds-endpoint.rds.amazonaws.com` with your actual endpoint
    - `admin` with your master username
    - You’ll be asked for the password (enter your RDS master password)

---

## 📌 2.4 - Load the Employee Data into RDS

### Step 1 - Confirm Connection Works
Once connected, you should see:
```bash
MySQL JS> \sql
```
That switches to SQL mode. You can run:
```sql
SHOW DATABASES;
```
You should see your database list. If your RDS was created without a DB, you can create one like this:
```sql
CREATE DATABASE employees;
```

---

### Step 2 - Load `employees.sql` Into RDS

The `employees.sql` file is in the `test_db` folder you cloned in step 2.1.

### Command:
```bash
mysql -h your-rds-endpoint.rds.amazonaws.com -u admin -p employees < employees.sql
```
- This will:
    - Create tables
    - Load all the employee data (300,024 records)

---

### Step 3 - Verify Data is Loaded
Login to RDS again:
```bash
mysqlsh -h your-rds-endpoint.rds.amazonaws.com -u admin -p
```
Switch to SQL mode:
```
MySQL JS> \sql
```
Select the employees database:
```sql
USE employees;
```
Check tables:
```sql
SHOW TABLES;
```
Check employee count:
```sql
SELECT COUNT(*) FROM employees;
```
You should see:
```
+----------+
| COUNT(*) |
+----------+
| 300024   |
+----------+
```
That means your data is correctly loaded. 🎉

---

## 📌 2.5 - Set Security Group to Open Port 3306 (Important)

### 1. Go to AWS EC2 Console (Security Groups)
- Open: [https://console.aws.amazon.com/ec2](https://console.aws.amazon.com/ec2)
- Find the **Security Group** attached to your RDS instance.

### 2. Edit Inbound Rules
- Add Rule:
    - **Type**: MySQL/Aurora
    - **Protocol**: TCP
    - **Port Range**: 3306
    - **Source**: Anywhere (0.0.0.0/0) — this is for demo only (for production you restrict it)

---

## 📌 2.6 - Final Test Connection from Your Machine
On your machine, test this final connection:
```bash
mysql -h your-rds-endpoint.rds.amazonaws.com -u admin -p
```
If it connects successfully, your setup is correct and the data is in place.

---

# ✅ Summary of Step 2 Checklist

✅ Clone the sample repo  
✅ Install MySQL Shell  
✅ Create employees database in RDS  
✅ Load `employees.sql` into RDS  
✅ Confirm 300,024 records exist  
✅ Open port 3306 in Security Group  
✅ Final test connection works

---


# ✅ **Step 3: Setup Cache Layer (Memcached)**

---

## 📌 3.1 - Why Memcached?  
- It’s **free** on AWS (within Free Tier for t3.micro).  
- It’s a simple key-value store, perfect for caching SQL query results (like employee data) to speed up response time.

---

## 📌 3.2 - Create ElastiCache Cluster (Memcached)

### 1. Open AWS Console
- Go to: [https://console.aws.amazon.com/elasticache](https://console.aws.amazon.com/elasticache)

### 2. Create Cluster
- Click **Create Cluster**
- Select: **Memcached**
- Cluster Name: `employee-memcached`
- Node Type: `t3.micro` (Free Tier eligible)
- Number of nodes: `1` (for now - scale later if needed)
- Subnet Group: Create new or select existing (should match RDS subnet group)
- Security Group: Use the same one used for RDS (this allows Lambda to talk to both)
- Port: `11211` (default for Memcached)

### 3. Review & Create
- After a few minutes, your Memcached endpoint will be available.

---

## 📌 3.3 - Open Port 11211 (Security Group)

### 1. Go to EC2 > Security Groups
- Find the security group attached to ElastiCache.

### 2. Edit Inbound Rules
- Add Rule:
    - **Type**: Custom TCP
    - **Protocol**: TCP
    - **Port Range**: 11211
    - **Source**: Anywhere (0.0.0.0/0) (for demo only — tighten this in production)

---

## 📌 3.4 - Connect to Memcached (Test Connection)

You can test using **telnet** (if your machine has it installed):
```bash
telnet your-memcached-endpoint.cache.amazonaws.com 11211
```
If connection is successful, you’ll see:
```
Trying X.X.X.X...
Connected to your-memcached-endpoint.cache.amazonaws.com.
Escape character is '^]'.
```
That confirms the cache is live.

---
