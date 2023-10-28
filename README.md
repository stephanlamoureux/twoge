# Twoge

Elon Musk, the renowned entrepreneur, is developing a new version of Twitter called "Twoge" where users are only allowed to tweet about Doge. While some may question the need for such an app. He has reached out to Codeplatoon to launch, maintain and deploy the app. 

## AWS Services and their purpose for Twoge

1. AWS EC2 (Elastic Compute Cloud) to host your twoge application.
2. AWS S3 (Simple Storage Service) to store your static files, such as images, videos, and other assets.
3. AWS IAM (Identity and Access Management) to manage service’s access and permissions to AWS resources.
4. AWS VPC (Virtual Private Cloud) to create a secure and isolated network environment for your application in the public subnet.
5. AWS ALB (Application Load Balancer) to distribute incoming traffic across multiple EC2 instances.
6. AWS ASG (Auto Scaling Group) to automatically scale your EC2 instances up or down based on the demand.
7. AWS SNS (Simple Notification Service) to receive notifications about your application's performance and health.
8. AWS RDS for database (or you can deploy a postgres database on a private subnet)
