# AWS 

## Access Types

* **Console access** – Administrators
* **AWS CLI access** – Admins / Developers
* **AWS SDK access** – Developers

## Learning Resources

* AWS Skill Builder
* AWS Events and Webinars

---

## Common Port Numbers

| Service     | Port     |
| ----------- | -------- |
| SSH         | 22       |
| HTTP        | 80       |
| HTTPS       | 443      |
| MySQL       | 3306     |
| Redis       | 6379     |
| MongoDB     | 27017    |
| DNS         | 53       |
| DHCP Server | 67 (UDP) |
| DHCP Client | 68 (UDP) |

### RDS Ports

| Database          | Port |
| ----------------- | ---- |
| Aurora (Postgres) | 5432 |
| Aurora (MySQL)    | 3306 |
| MySQL / MariaDB   | 3306 |
| PostgreSQL        | 5432 |
| SQL Server        | 1433 |
| Oracle            | 1521 |

---

## Creating an IAM User

1. Go to **IAM Service**
2. Create user
3. Username – `thub` (I want to create IAM user)
4. Set custom password → Next → Next
5. Open IAM user in **Incognito window**
6. Launch EC2 instance
7. Go to **Users → Permissions**
8. Add permissions → Attach policies directly
9. Select **AdministratorAccess** → Next

---

## Installing AWS CLI (Linux)

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```

---

## Sample IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowGroupToSeeBucketListInTheConsole",
      "Action": ["s3:ListAllMyBuckets"],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::*"]
    }
  ]
}
```

---

## Task 1 – Creating IAM & S3 Using CLI

### IAM Commands

```bash
aws iam create-user --user-name NAME   --> Creates IAM User
aws iam create-login-profile --user-name NAME --password 'Aditya@123' --no-password-reset-required   --> Sets password for the user
aws sts get-caller-identity
aws iam delete-login-profile --user-name NAME   --> Deletes the login profile
aws iam delete-user --user-name NAME   --> Deletes the user
aws iam create-group --group-name NAME   --> Creates the group
aws iam add-user-to-group --user-name NAME --group-name NAME   --> Adds group to the user
```

### S3 Commands

```bash
aws s3api create-bucket --bucket BUCKETNAME --region us-east-1   --> Creates a bucket
aws s3api put-object --bucket BUCKETNAME --key Development/project1.pdf   --> Upload the directories into bucket
```

### Group Policy (iampolicy.json)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowGroupToSeeBucketListAndAlsoAllowGetBucketLocationRequiredForListBucket",
      "Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::*"]
    },
    {
      "Sid": "AllowRootLevelListingOfBucket",
      "Action": ["s3:ListBucket"],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::BUCKETNAME"],
      "Condition": {
        "StringEquals": {
          "s3:prefix": [""],
          "s3:delimiter": ["/"]
        }
      }
    }
  ]
}
```

```bash
aws iam create-policy --policy-name GroupPolicy --policy-document file://iampolicy.json   --> Creates a policy with the name GroupPolicy
aws iam attach-group-policy --policy-arn arn:aws:iam::AWSACCOUNTID:policy/GroupPolicy --group-name dev   Attaches policy to the group, so that the both users get the access to s3 bucket
```

---

## Task 2 – IAM Role with EC2 (No Access Keys)

* Launch Ubuntu EC2
* Install AWS CLI
* Create S3 bucket
* Create IAM Role with **S3FullAccess**
* Attach role to EC2

```bash
aws s3 ls
aws ec2 describe-instances
```

```bash
aws s3 cp file1 s3://bucketname
rm file1
aws s3 cp s3://bucketname/file1 .
aws s3 rm s3://bucketname/file1
```

---

## Task 3 – STS Assume Role (Least Privilege)

* Create S3 bucket
* Create user `alice`
* Create role with **S3FullAccess**
* Attach **inline STS AssumeRole policy** to user
* User switches role to access S3

**Notes**:

* User → only inline policy
* Role → only permission policy
* STS gives temporary access

---

## Task 4 – Cross Account Role

* Create role (ReadOnlyAccess)
* Create user with AdminAccess
* Use **Switch Role** URL

**Important**:

* Root account cannot assume role
* Only IAM users can switch role

---

## Task 5 – Elastic Beanstalk Deployment

### Roles

**Instance Profile Role** – `EBSprofilerole`

* AWSElasticBeanstalkWebTier
* AWSElasticBeanstalkWorkerTier
* AWSElasticBeanstalkMulticontainerDocker

**Service Role** – `EBSservicerole`

* AWSElasticBeanstalkEnhancedHealth
* AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy

Deploy using **Tomcat platform** for HTML/CSS

---

## Task 6 – Lambda Automation

### Stop EC2 Instance

```python
import boto3
region = 'us-east-1'
instances = ['i-xxxxxxxx']
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.stop_instances(InstanceIds=instances)
```

### Create Snapshot

```python
import boto3, datetime
ec2 = boto3.client('ec2')
VOLUME_IDS = ['vol-xxxxxxxx']

def lambda_handler(event, context):
    for volume_id in VOLUME_IDS:
        timestamp = datetime.datetime.utcnow().strftime('%Y-%m-%d-%H-%M-%S')
        ec2.create_snapshot(VolumeId=volume_id)
```

---

## Task 7 – Scaling

### Vertical Scaling

* Change instance type (manual)

### Horizontal Scaling

* AMI → Launch Template → Auto Scaling Group
* CPU based scaling

---

## Task 8 – EC2 + MySQL Login System

* Install MySQL
* Create DB and users table
* Install PHP backend
* Connect frontend and backend

---

## Task 9 – EC2 + RDS Login System

* Create RDS MySQL
* Connect via endpoint
* Deploy PHP backend
* Store credentials in RDS

---

## Task 10 – ElastiCache (Redis)

* Redis OSS
* Port 6379
* In-memory cache

---

## Docker Basics

* Images → Containers
* Docker Hub
* Port mapping

```bash
docker pull nginx
docker run -d -p 8080:80 nginx
```

---

## Demo Project

* Node.js app on VM
* MongoDB in Docker container
* mongo-express UI

---

## Elastic Container Registry (ECR)

* Create repository
* Login via AWS CLI
* Push Docker images

---

## SSL & Domains

* AWS Certificate Manager
* Let’s Encrypt
* certbot

---

## DevOps Tools

* Slack
* Jira
* Standup Meetings
* Microsoft Planner
