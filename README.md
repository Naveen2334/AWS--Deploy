Here’s a **step-by-step guide for Java backend developers** on how to deploy a web application in **AWS**. We'll use **Spring Boot + AWS EC2 + RDS + S3 + Route53 + ACM + Load Balancer (if needed)**. This is written for clarity and real-world deployment.

---

## ✅ Prerequisites

Before starting:

* Java backend application (e.g. Spring Boot)
* AWS Account
* Maven/Gradle installed
* MySQL/PostgreSQL schema (if DB is used)
* Domain name (if custom domain is needed)

---

## 🌐 STEP 1: Package Your Java Application

### ✅ Create executable JAR

```bash
./mvnw clean package
```

Output: `target/myapp.jar`

---

## ☁️ STEP 2: Create an EC2 Instance (Linux)

1. Go to **EC2 Console** → Launch Instance
2. Choose AMI: **Amazon Linux 2** or **Ubuntu 22.04**
3. Instance Type: `t2.micro` (Free Tier) or as needed
4. Key Pair: Create/download a `.pem` file
5. Security Group:

   * Allow **port 22** (SSH)
   * Allow **port 80** (HTTP)
   * Allow **port 8080** (if running directly)
6. Storage: Minimum 8 GB
7. Launch Instance

---

## 🔐 STEP 3: SSH into EC2 & Install Java

```bash
ssh -i mykey.pem ec2-user@<EC2_PUBLIC_IP>
```

### For Amazon Linux:

```bash
sudo yum update -y
sudo amazon-linux-extras enable corretto11
sudo yum install java-11-amazon-corretto -y
```

### For Ubuntu:

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

---

## 📁 STEP 4: Copy JAR File to EC2

From your local machine:

```bash
scp -i mykey.pem target/myapp.jar ec2-user@<EC2_PUBLIC_IP>:/home/ec2-user
```

---

## ▶️ STEP 5: Run the Spring Boot Application

```bash
nohup java -jar myapp.jar > log.txt 2>&1 &
```

Your app will now run in the background.

---

## 🌐 STEP 6: Configure Domain (Optional, with Route 53)

1. Buy or transfer a domain in Route 53
2. Go to Hosted Zones → Add A Record
3. Point it to your EC2 public IP or Load Balancer

---

## 🔒 STEP 7: Set Up HTTPS with SSL (ACM + ALB Method)

### Option 1: Use Load Balancer (HTTPS via ACM)

1. Go to **EC2 → Load Balancer**
2. Create **Application Load Balancer**
3. Target group → Add your EC2 instance
4. Add HTTPS listener
5. Get SSL certificate from **ACM** (Amazon Certificate Manager)
6. Forward HTTPS to port 8080

---

## 🗄️ STEP 8: Use AWS RDS (Optional for MySQL/PostgreSQL)

1. Go to **RDS Console**
2. Create new DB instance
3. Public access: Enable (if needed)
4. Note username, password, and endpoint
5. Add inbound rule to EC2 Security Group → Allow RDS port (e.g. 3306)

### In `application.properties`:

```properties
spring.datasource.url=jdbc:mysql://<rds-endpoint>:3306/dbname
spring.datasource.username=admin
spring.datasource.password=password
```

---

## ☁️ STEP 9: Use S3 for File Upload (Optional)

1. Go to **S3** → Create Bucket
2. Enable public access (or IAM-based)
3. Note bucket name

### Add IAM Role to EC2:

* Go to IAM → Create Role → EC2 + S3 Full Access
* Attach role to EC2 instance

### In Java Code:

Use AWS SDK (v2 or v1) to upload files.

---

## 🔄 STEP 10: Automate with Systemd (Optional)

To keep app alive on reboot:

```bash
sudo vim /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Spring Boot App
After=network.target

[Service]
User=ec2-user
ExecStart=/usr/bin/java -jar /home/ec2-user/myapp.jar
SuccessExitStatus=143
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reexec
sudo systemctl start myapp
sudo systemctl enable myapp
```

---

## 📊 STEP 11: Monitoring & Logs

```bash
tail -f log.txt
```

Or integrate with:

* **CloudWatch**
* **ELK stack**
* **Prometheus + Grafana**

---

## ✅ Bonus: Security Best Practices

* Change default SSH port (optional)
* Enable fail2ban or AWS GuardDuty
* Use Secrets Manager for DB credentials
* Always use IAM roles instead of hardcoded credentials
* Restrict security group rules to known IPs

---

## 🧠 Summary

| Step | Description                        |
| ---- | ---------------------------------- |
| 1    | Package Spring Boot JAR            |
| 2    | Launch EC2                         |
| 3    | Install Java                       |
| 4    | Copy JAR                           |
| 5    | Run JAR                            |
| 6    | (Optional) Set up Route53 + Domain |
| 7    | (Optional) HTTPS via Load Balancer |
| 8    | (Optional) Use RDS DB              |
| 9    | (Optional) Use S3 for uploads      |
| 10   | Auto-start using Systemd           |
| 11   | Monitor logs and setup alerts      |

---

If you'd like, I can also:

* Prepare a **flowchart** of the process
* Provide a **Terraform or Ansible script** for automation
* Help with **CI/CD setup using GitHub Actions**

Just let me know!
<img width="1024" height="1536" alt="ChatGPT Image Jul 30, 2025, 12_34_56 PM" src="https://github.com/user-attachments/assets/728ba922-2974-4f60-8fb5-ce31899e8a48" />
