# Twoge Deployment Guide

- [Twoge Deployment Guide](#twoge-deployment-guide)
	- [Twoge](#twoge)
		- [Requirements](#requirements)
	- [VPC (*Virtual Private Cloud*)](#vpc-virtual-private-cloud)
		- [Create a VPC](#create-a-vpc)
		- [Enable DNS hostnames and resolution](#enable-dns-hostnames-and-resolution)
	- [Subnets](#subnets)
		- [Subnet 1](#subnet-1)
		- [Subnet 2](#subnet-2)
	- [Internet Gateways](#internet-gateways)
		- [Create Internet Gateway](#create-internet-gateway)
		- [Attach gateway to the VPC](#attach-gateway-to-the-vpc)
	- [Route Tables](#route-tables)
		- [Create Route Table](#create-route-table)
		- [Create Routes](#create-routes)
		- [Subnet Associations](#subnet-associations)
	- [RDS (*Relational Database Service*)](#rds-relational-database-service)
		- [Create a RDS database](#create-a-rds-database)
		- [Connect to the database using pgAdmin4](#connect-to-the-database-using-pgadmin4)
	- [EC2 (*Elastic Compute Cloud*)](#ec2-elastic-compute-cloud)
		- [Create a new EC2 instance](#create-a-new-ec2-instance)
		- [Create a security group](#create-a-security-group)
	- [S3](#s3)
		- [Create a bucket](#create-a-bucket)
		- [IAM Role](#iam-role)
		- [Permissions](#permissions)
		- [Attach S3 role](#attach-s3-role)
		- [Bucket Policy](#bucket-policy)
		- [Test S3 Access](#test-s3-access)
	- [Load Balancing](#load-balancing)
		- [Security Group](#security-group)
		- [Target Group](#target-group)
		- [Application Load Balancer](#application-load-balancer)
	- [Auto Scaling](#auto-scaling)
		- [Launch template](#launch-template)
		- [Auto Scaling Group](#auto-scaling-group)
		- [ASG Policy](#asg-policy)
	- [Future Improvements](#future-improvements)
		- [Database](#database)
		- [S3](#s3-1)
		- [Diagram](#diagram)
		- [README](#readme)

## Twoge

<div align="center">
 <img
  width="100"
  alt="Project Twoge"
  src="./img/twoge.png" />
</div>

<br>

Twoge is a social media platform dedicated solely to tweets about Dodge. This application is built using Flask, SQLAlchemy, and PostgreSQL.

The repository for it is located [here](https://github.com/chandradeoarya/twoge).

### Requirements

1. Create an Amazon VPC with two public subnets.

2. Host the static files like images and videos on an S3 bucket.

3. Create an IAM role that allows public access to the S3 bucket.

4. Launch an EC2 instance with an Amazon Linux 2 AMI, using the IAM role you previously created.

5. Install and configure the twoge application on an EC2 instance.

6. Create an Amazon ALB and configure it to route traffic to your EC2 instance. Add a listener rule to forward traffic to HTTP.

7. Create an Amazon ASG that automatically launches EC2 instances when traffic to your application exceeds a certain threshold. Configure the ASG to use the Amazon ALB as the load balancer.

8. Use Amazon SNS to receive notifications when the number of EC2 instances in your ASG increases or decreases. Configure an SNS topic to send email notifications to your email address. Stop a server and SNS should send email notifications about the server shutdown.

9. Run the instance Python stress script and show the ASG in play.

## VPC (*Virtual Private Cloud*)

### Create a VPC

1. Resources to create: VPC only
2. Name: twoge-vpc
3. IPv4 CIDR: 10.0.0.0/24
4. The rest is default

### Enable DNS hostnames and resolution

1. Select VPC
2. Click Actions
3. Click Edit VPC settings
4. Enable DNS hostnames and DNS resolution

## Subnets

### Subnet 1

1. Create subnet
2. VPC ID: Select VPC (twoge-vpc)
3. Subnet name: public-1a
4. Availability Zone: 1a
5. IPv4 subnet CIDR block: 10.0.0.0/25
6. The rest is default

### Subnet 2

1. Create subnet
2. VPC ID: Select VPC (twoge-vpc)
3. Subnet name: public-1b
4. Availability Zone: 1b
5. IPv4 subnet CIDR block - 10.0.0.128/25

Created CIDR blocks using [Subnet Calculator](https://www.davidc.net/sites/default/subnets/subnets.html).

## Internet Gateways

To allow internet access to the twoge VPC.

### Create Internet Gateway

1. Create internet gateway
2. Name: twoge-igw
3. Click Create

### Attach gateway to the VPC

1. Select the twoge IGW
2. Actions -> Attach to twoge VPC

## Route Tables

### Create Route Table

1. Create route table
2. Name: twoge-route
3. VPC: Select twoge-vpc
4. Create route table

### Create Routes

1. Select twoge-route
2. Select Routes tab
3. Click Edit routes
4. Add route
5. Destination: 0.0.0.0/0
6. Target: Internet Gateway -> twoge-igw
7. Save

### Subnet Associations

1. Select the twoge-route
2. Click Subnet associations tab -> Edit subnet associations
3. Select the public-1a and public-1b subnets
4. Save associations

## RDS (*Relational Database Service*)

### Create a RDS database

1. Create database
2. Creation method: Standard create
3. Engine type: PostgreSQL
4. Version: Latest version
5. Templates: Free tier
6. DB Instance ID: twoge-db
7. Create username/password
8. VPC: Select the twoge VPC
9. Public access: yes
10. Remaining options are left at default

### Connect to the database using pgAdmin4

1. hostname: RDS database endpoint
2. port: 5432
3. user: postgres
4. password: the one made during RDS creation

## EC2 (*Elastic Compute Cloud*)

### Create a new EC2 instance

1. Launch instance
2. Name: twoge
3. AMI: Amazon Linux 2
4. Instance type: t2.micro
5. Key pair: Choose or create your key pair
6. Network: Select the twoge vpc
7. Security group: Create a SG with the inbound rules below
8. Auto-assign public ip: Enabled
9. User data: Add the shell script below

### Create a security group

Inbound rules:

```
HTTP | 80 | 0.0.0.0/0
HTTPS | 443 | 0.0.0.0/0
SSH | 22 | 0.0.0.0/0
PostgreSQL | 0.0.0.0/0
Custom TCP | 9876 | 0.0.0.0/0
```

Add the following script to the user data section:

```sh
#!/bin/bash
sudo yum update -y
sudo yum install git python3-pip -y
git clone https://github.com/chandradeoarya/twoge
cd twoge
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt

echo 'SQLALCHEMY_DATABASE_URI = "postgresql://username:password@endpoint/database"' > .env

echo '
Description=Gunicorn instance to serve twoge

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
WorkingDirectory=/home/ec2-user/twoge
Environment="PATH=/home/ec2-user/twoge/venv/bin"
ExecStart=/home/ec2-user/twoge/venv/bin/gunicorn app:app -c /home/ec2-user/twoge/gunicorn_config.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target' > twoge.service

sudo cp twoge.service /etc/systemd/system/twoge.service
sudo systemctl daemon-reload
sudo systemctl enable --now twoge
sudo systemctl status twoge
```

## S3

### Create a bucket

1. Bucket name: stephan-twoge
2. Region: eu-central-1
3. Remaining settings are left at default

Next, upload the image files from the 'img' directory in the twoge app.

### IAM Role

An IAM role must be created to allow access to an S3 bucket from an EC2 instance.

1. IAM Dashboard
2. Roles
3. Create role
4. Entity type: AWS service
5. Use case: Select EC2
6. Click next

### Permissions

1. Search for S3
2. Select AmazonS3FullAccess
3. Click Next
4. Give a name and description
5. Create role

### Attach S3 role

1. Go to the EC2 Instance
2. Click Actions -> Security -> Modify IAM role
3. Select s3access role
4. Update

### Bucket Policy

1. Select the twoge bucket
2. Click the Permissions tab
3. Under bucket policy, click edit.
4. Enter the policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::your-bucket-name",
                "arn:aws:s3:::your-bucket-name/*"
            ]
        }
    ]
}
```

### Test S3 Access

```sh
aws s3 ls s3://your-bucket-name/
```

## Load Balancing

### Security Group

Create a security group to be used by the load balancer.

Inbound rules:

```
Name: twogeALBSG
HTTP | TCP | 80  0.0.0.0/0
HTTPS | TCP | 443 | 0.0.0.0/0
Custom TCP | TCP | 9876 | 0.0.0.0/0
```

### Target Group

You must first create a group so the load balancer knows where to send the traffic.

1. Target type: Instances
2. Group name: twoge-tg
3. Protocol: HTTP Port 9876
4. IP type: IPv4
5. VPC: twoge VPC
6. Protocol version: HTTP1
7. Health checks: HTTP
8. Next
9. Select the twoge EC2 instance, click include as pending below
10. Create target group

### Application Load Balancer

1. Load balancer -> Create -> Application load balancer
2. Name: twoge-alb
3. Scheme: Internet-facing
4. IP type: IPv4
5. VPC: twoge vpc
6. Mappings: select both subnets
7. Security groups: twogeALBSG
8. Listener: HTTP Port 80 -> Select twoge target group
9. Create

## Auto Scaling

### Launch template

We must first create a launch template that the ASG will use to create new instances.

1. Name: twoge-template
2. Select key pair
3. AMI: twoge
4. Security group: twoge
5. Advanced details -> IAM instance profile -> s3access

### Auto Scaling Group

1. Create
2. Name: twoge-asg
3. vCPU and memory: min 0, max 50
4. VPC: twoge-vpc
5. AZ and subnets: select both
6. Attach to an existing load balancer
7. Select twoge target group
8. Turn on ELB health checks
9. Group size: desired 2, min 1, max 3.
10. Add notification -> create a name and add email
11. Confirm and create

### ASG Policy

1. Select ASG
2. Create dynamic scaling policy
3. Simple scaling
4. Create cloudwatch alarm
   1. Select metric -> EC2
   2. By auto scaling group -> select twoge ASG
   3. Select twoge-asg metric value of CPUUtilization -> select metric
   4. Threshold(static) greater/equal -> 50
   5. Notifications -> Create new topic
   6. Topic name -> add email -> create topic
   7.  Create alarm name -> Create alarm
5. Return to creating the dynamic scaling policy, refresh the cloudwatch alarm, and select the one just created.
6. Take Action: Add 1 capacity units
7. Wait 60 seconds
8. Create
9. Confirm email to subscribe

## Future Improvements

### Database

- Private subnet for the PostgreSQL database
- Add a NAT gateway for internet access
- Create a bastion host to tunnel into the private instance

### S3

- Serve the image files for the app through an S3 bucket

### Diagram

Add more layers to show lower-level details.

- Subnets
- Routing
- Instance
- RDS

### README

- Add screenshots
