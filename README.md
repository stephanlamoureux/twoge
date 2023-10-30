# Deployment Guide

- [Deployment Guide](#deployment-guide)
	- [VPC (*Virtual Private Cloud*)](#vpc-virtual-private-cloud)
	- [EC2 (*Elastic Compute Cloud*)](#ec2-elastic-compute-cloud)
	- [RDS (*Relational Database Service*)](#rds-relational-database-service)
	- [AWS Services \& their purpose for Twoge](#aws-services--their-purpose-for-twoge)

## VPC (*Virtual Private Cloud*)

Create a VPC with two public subnets.

## EC2 (*Elastic Compute Cloud*)

1. Create a new EC2 instance:

```sh
Amazon Linux 2
t2.micro
choose key pair
allow SSH and HTTP
```

Add the following script to the user data section:

```sh
#!/bin/bash
sudo yum update -y
sudo yum install git -y
git clone https://github.com/chandradeoarya/twoge
cd twoge
sudo yum install python-pip -y
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

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
sudo systemctl enable twoge
sudo systemctl start twoge
sudo systemctl status twoge

sudo amazon-linux-extras install nginx1 -y
```

sudo nano /etc/nginx/sites-available/app

## RDS (*Relational Database Service*)

1. Create a new RDS database:

```sh
Standard create
PostgreSQL
Latest version
Free tier
DB Instance ID
Create username/password
Select your VPC
Public access
```

After the database is ready, edit the default security group to add an inbound rule for PostgreSQL.

Connect to the database using pgAdmin4:

```sh
hostname: RDS database endpoint
port: 5432
user: postgres
password: the one made during RDS creation
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
