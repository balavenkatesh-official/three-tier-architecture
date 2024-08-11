Here's the complete content formatted for a GitHub README:

```markdown
# Project Setup

This project involves setting up a VPC with multiple subnets, security groups, and launching an EC2 instance for a web application. The EC2 instance will connect to an RDS database, run a Node.js application using PM2, and be configured with Auto Scaling and an Application Load Balancer.

## VPC Configuration

- **VPC Name:** project-vpc
- **CIDR Block:** 10.0.0.0/16

### Subnets

- **private-DB-subnet-AZ-1:** 10.0.1.0/24
- **public-web-subnet-AZ-1:** 10.0.2.0/24
- **private-App-subnet-AZ-2:** 10.0.3.0/24
- **private-App-subnet-AZ-1:** 10.0.4.0/24
- **public-web-subnet-AZ-2:** 10.0.5.0/24
- **private-DB-subnet-AZ-2:** 10.0.6.0/24

### RDS Subnet Group

- **Name:** edbence-rds-subnet-group
- **Availability Zones:** us-east-1a, us-east-1b

### Route Tables

- **Public Route Table:** edbence-public-rt
- **Private Route Table AZ-1:** edbence-private-rt-az1
- **Private Route Table AZ-2:** edbence-private-rt-az2

### Internet Gateway

- **Name:** edbence-igw

### NAT Gateways

- **NAT Gateway AZ-1:** edbence-nat-az1
- **NAT Gateway AZ-2:** edbence-nat-az2

### Security Groups

- **internet-facing-sg:** Port 80 (all IPv4)
- **web-tier-sg:** Port 80 (all IPv4), inbound from internet-facing-sg
- **internal-LB:** Port 80, inbound from web-tier-sg
- **app-tier-sg:** Port 4000 (all IPv4), inbound from internal-LB
- **DB-tier-sg:** Port 3306, inbound from app-tier-sg

## EC2 Instance Setup

1. **Launch Instance**
   - **AMI:** Amazon Linux 2
   - **Subnet:** private-App-subnet-AZ-1
   - **IAM Role:** s3-ssm-access

2. **Install MySQL Client**
   ```bash
   sudo yum install mysql -y
   mysql -h database-1.cbayq8eeu3h9.ap-south-1.rds.amazonaws.com -u admin -p
   ```

3. **Create Database and Table**
   ```sql
   CREATE DATABASE webappdb;
   SHOW DATABASES;
   USE webappdb;

   CREATE TABLE IF NOT EXISTS transactions(
       id INT NOT NULL AUTO_INCREMENT, 
       amount DECIMAL(10,2), 
       description VARCHAR(100), 
       PRIMARY KEY(id)
   );

   SHOW TABLES;
   INSERT INTO transactions (amount, description) VALUES ('400', 'groceries');
   SELECT * FROM transactions;
   ```

4. **Download and Configure Application**
   ```bash
   aws s3 cp s3://edbence-project1/app-tier/ app-tier --recursive

   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
   source ~/.bashrc
   nvm install 16
   nvm use 16
   npm install -g pm2

   cd ~/app-tier
   npm install
   pm2 start index.js
   pm2 list
   pm2 logs
   pm2 startup
   pm2 save
   ```

5. **Test Application**
   ```bash
   curl http://localhost:4000/health
   curl http://localhost:4000/transaction
   ```

6. **Create AMI**
   - **AMI Name:** APP-AMI

## Load Balancer and Auto Scaling Setup

1. **Create Target Group**
   - Set up a target group that will register your EC2 instances to distribute traffic evenly.
   - Choose `HTTP` as the protocol and port `80`.
   - Select the VPC `project-vpc`.
   - Choose the target type as `Instance`.
   - Configure the health check settings.

2. **Create Application Load Balancer**
   - Navigate to the EC2 console and select `Load Balancers`.
   - Choose `Create Load Balancer` and select `Application Load Balancer`.
   - Provide a name for the load balancer.
   - Select `Internet-facing` as the scheme.
   - Choose the VPC `project-vpc`.
   - Select the appropriate subnets (public-web-subnet-AZ-1 and public-web-subnet-AZ-2).
   - Attach a security group (internet-facing-sg) to allow traffic on port 80.
   - Register your target group created in the previous step.

3. **Create Launch Template**
   - Navigate to the EC2 console and select `Launch Templates`.
   - Choose `Create Launch Template`.
   - Provide a name for the launch template.
   - Specify the AMI created earlier (`APP-AMI`).
   - Choose the instance type (e.g., t2.micro).
   - Select the appropriate subnet (private-App-subnet-AZ-1).
   - Configure security groups (app-tier-sg).
   - Add user data if needed for instance initialization.

4. **Create Auto Scaling Group**
   - Navigate to the EC2 console and select `Auto Scaling Groups`.
   - Choose `Create Auto Scaling Group`.
   - Provide a name for the auto-scaling group.
   - Select the launch template created in the previous step.
   - Choose the VPC `project-vpc`.
   - Select the subnets (private-App-subnet-AZ-1 and private-App-subnet-AZ-2).
   - Attach the Application Load Balancer created earlier.
   - Configure the desired capacity, minimum, and maximum number of instances.
   - Set up scaling policies if needed.
   - Review and create the auto-scaling group.

Refer to the following [AWS Workshop Guide](https://catalog.us-east-1.prod.workshops.aws/workshops/85cd2bb2-7f79-4e96-bdee-8078e469752a/en-US/part4/targetgroup) for detailed steps.
```

This README includes all the steps and commands required to set up the project, including the infrastructure, EC2 instance, database, application setup, and auto-scaling configuration.
