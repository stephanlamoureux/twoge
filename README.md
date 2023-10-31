# Deployment Guide

- [Deployment Guide](#deployment-guide)
	- [VPC (*Virtual Private Cloud*)](#vpc-virtual-private-cloud)
		- [Subnets](#subnets)
		- [Internet Gateway](#internet-gateway)
		- [Route Table](#route-table)
		- [Subnet Associations](#subnet-associations)
	- [RDS (*Relational Database Service*)](#rds-relational-database-service)
	- [EC2 (*Elastic Compute Cloud*)](#ec2-elastic-compute-cloud)
	- [S3](#s3)
	- [IAM Role](#iam-role)
	- [AWS Services \& their purpose for Twoge](#aws-services--their-purpose-for-twoge)

## VPC (*Virtual Private Cloud*)

Create a VPC:

```sh
VPC only
Name - twoge-vpc
IPv4 CIDR - 10.0.0.0/24
```

Enable DNS hostnames and resolution:

1. Select VPC
2. Click Actions
3. Click Edit VPC settings
4. Enable DNS hostnames and DNS resolution

### Subnets

Create subnets:

Subnet 1:

```sh
Select VPC (twoge-vpc)
Subnet name - public-1a
Availability Zone - 1a
IPv4 subnet CIDR block - 10.0.0.0/25
```

Subnet 2:

```sh
Select VPC (twoge-vpc)
Subnet name - public-1b
Availability Zone - 1b
IPv4 subnet CIDR block - 10.0.0.128/25
```

[Subnet Calculator](https://www.davidc.net/sites/default/subnets/subnets.html)

### Internet Gateway

To allow internet access to the twoge VPC.

Create internet gateway:

1. Name - twoge-igw
2. Create
3. Attach to twoge VPC

### Route Table

Create route table:

1. Name - twoge-route
2. Select twoge-vpc
3. Create

Routes:

1. Select twoge-route
2. Select Routes tab
3. Click Edit routes
4. Add route
5. Destination - 0.0.0.0/0
6. Target - Internet Gateway and then select the twoge-igw
7. Save

### Subnet Associations

1. Edit subnet associations
2. Select the public-1a and public-1b subnets
3. Save associations

## RDS (*Relational Database Service*)

1. Create a new RDS database:

```sh
Standard create
PostgreSQL
Latest version
Free tier
DB Instance ID - twoge-db
Create username/password
Select the twoge VPC
Public access - yes
Remaining options are left at default
```

After the database is ready, edit the default security group to add an inbound rule for PostgreSQL.

Connect to the database using pgAdmin4:

```sh
hostname: RDS database endpoint
port: 5432
user: postgres
password: the one made during RDS creation
```

## EC2 (*Elastic Compute Cloud*)

1. Create a new EC2 instance:

```sh
Amazon Linux 2
t2.micro
choose key pair
select the twoge vpc
auto-assign public ip

Create a security group

Inbound rules:

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

echo 'SQLALCHEMY_DATABASE_URI = "postgresql://postgres:password@endpoint/postgres"' > .env

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

Create a bucket:

```sh
Bucket name - stephan-twoge
Region - eu-central-1
Remaining settings are left at default
```

Next upload the image files from the 'img' directory in the twoge app.

## IAM Role

An IAM role must be created to allow access to an S3 bucket from an EC2 instance.

Create an IAM Role:

1. IAM Dashboard
2. Roles
3. Create role
4. Entity type - AWS service
5. Use case - Select EC2
6. Click next

Permissions:

1. Search for S3
2. Select AmazonS3FullAccess
3. Click Next
4. Give a name and description
5. Create role

Attach S3 role:

1. Go to the EC2 Instance
2. Click Actions -> Security -> Modify IAM role
3. Select s3access role
4. Update

Bucket Policy:

1. Select the twoge bucket
2. Click Permissions tab
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

<hr>

## AWS Services & their purpose for Twoge

- [AWS EC2 (*Elastic Compute Cloud*)](https://aws.amazon.com/ec2/) - Host the application.
- [AWS S3 (*Simple Storage Service*)](https://aws.amazon.com/s3/) - Store static files, such as images, videos, and other assets.
- [AWS IAM (*Identity & Access Management*)](https://aws.amazon.com/iam/) - Manage access and permissions to AWS resources.
- [AWS VPC (*Virtual Private Cloud*)](https://aws.amazon.com/vpc/) - Create a secure and isolated network environment.
- [AWS ALB (*Application Load Balancer*)](https://aws.amazon.com/alb/) - Distribute incoming traffic across multiple EC2 instances.
- [AWS ASG (*Auto Scaling Group*)](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html) - Automatically scale EC2 instances up or down based on the demand.
- [AWS SNS (*Simple Notification Service*)](https://aws.amazon.com/sns/) - Receive notifications about the app's performance & health.
- [AWS RDS (*Relational Database Service*)](https://aws.amazon.com/rds/) - PostgreSQL database.

<br>
<br>

<div align="center">
 <img
  width="100"
  alt="Project Twoge"
  src="./img/twoge.png" />
 <h3>Twoge</h3>
 <p>A social media platform dedicated solely to tweets about Dodge.</p>
</div>

<div align="center">
 <img
  width="500"
  alt="Project Twoge"
  src="./img/twoge-cover.png" />
 <p>This application is built using Flask, SQLAlchemy, and PostgreSQL.</p>
</div>
