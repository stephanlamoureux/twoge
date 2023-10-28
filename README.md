# Twoge

<div align="center">
 <img
  width="100"
  alt="Project Twoge"
  src="./img/twoge.png" />
 <p>Twoge is a social media platform dedicated solely to tweets about Dodge.</p>
</div>

<div align="center">
 <img
  width="500"
  alt="Project Twoge"
  src="./img/twoge-cover.png" />
 <p>This application is built using Flask, SQLAlchemy, and PostgreSQL.</p>
</div>

## AWS Services and their purpose for Twoge

1. AWS EC2 (Elastic Compute Cloud) to host the application.
2. AWS S3 (Simple Storage Service) to store static files, such as images, videos, and other assets.
3. AWS IAM (Identity and Access Management) to manage service’s access and permissions to AWS resources.
4. AWS VPC (Virtual Private Cloud) to create a secure and isolated network environment.
5. AWS ALB (Application Load Balancer) to distribute incoming traffic across multiple EC2 instances.
6. AWS ASG (Auto Scaling Group) to automatically scale EC2 instances up or down based on the demand.
7. AWS SNS (Simple Notification Service) to receive notifications about the application's performance and health.
8. AWS RDS (Relational Database Service) for the database.

```sh
sudo apt update -y

sudo apt install git python3 -y

sudo apt install python3-pip -y

git clone https://github.com/chandradeoarya/twoge.git

cd twoge

python3 -m venv venv

source venv/bin/activate

pip install -r requirements.txt
```

Create a .env file:

```sh
SQLALCHEMY_DATABASE_URI = "PostgreSQL database URL"
```

Create a twoge.service file:

```sh
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
WantedBy=multi-user.target'
```

```sh
sudo cp twoge.service /etc/systemd/system/twoge.service

sudo systemctl daemon-reload

sudo systemctl enable twoge

sudo systemctl start twoge

sudo systemctl status twoge
```
