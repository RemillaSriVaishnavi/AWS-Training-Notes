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
* Don't create any access key
* Connect it to the MobaXterm
* Install AWS CLI
* Create S3 bucket
* Create IAM Role with **S3FullAccess**
* Attach role to EC2

```bash
aws s3 ls   --> List the buckets created
aws ec2 describe-instances   --> Shows an error
```

```bash
- Create file using cat command
aws s3 cp file1 s3://bucketname   --> Copy that file to the bucket
rm file1   --> Remove the file in ec2 instance
aws s3 cp s3://bucketname/file1 .   --> The file from s3 is copy backed to the ec2 instance
aws s3 rm s3://bucketname/file1   --> Completely deletes the file from bucket. But still the file will be in ec2 instance
```

---

## Task-3:

This task is about securely managing access to AWS resources using IAM users, roles, policies, and STS (Security Token Service).It demonstrates how to restrict a user’s access to a specific service (S3) via role assumption, following the principle of least privilege.

1. Initially create a bucket
2. Create an user 'alice'
3. Create role -- aws account -- This account -- Permissions(s3fullaccess) -- give any role name(iam-s3-alice-user)
4. Go to the user created and add policy
   Add permission -- create Inline policy -- service is STS and permissions(write - Assumerole) -- Add ARN to it(Role arn) --
   create Policy and give any name to the policy(S3AlicePolicy)
5. Now the policy is attached to the user(The policy is customer inline type)

⦁	User should have only inline policy and no other permissions
⦁	Role should have only permissions and no other policies

6. Go to incognito window and login with the user
7. When we login with the user we don't have access to the s3 service. So we need to switch the role
8. Switch role
9. After switching the role we get acces to the s3 bucket and we don't get access to the ec2 service because while creating a role we gave only s3 access permission.

⦁	The Inline Policy attached to the user have strict permissions and the policy is applicable to the particular one user only.
⦁	ARN - AWS Resourse Number
⦁	STS - Security Token Service --> It gives temporary access and cross account access

---

## Task-4:

1. Create role
   AWS account -- Another account(Here give other's ID) -- Give just 'ReadOnlyAccess' Policy
2. Create one user(alice) -- Give administrator access to the user in permissions
3. Open incognito window and login with user created
4. Open next incognito tab
5. Another account user should copy this and give to me --> Go to roles, select and open role and copy the 'Link to switch roles in console link
6. I should paste the link in the incognito tab and switch role
7. Now I will be able to access the Another account but I will be just able to see the services and and I acnnot edit or delete anything in that account.

⦁	Root account cannot assume the role.
⦁	Any other IAM user should switch the role.

---

## Task-5:

Deploying a website into cloud using Elastic BeanStalk:

1. Create 2 roles
   ### (I) Role name(EBSprofilerole)
   AWS service -- Elastic BeanStalk(Use case) -- Elastic Beanstalk - Compute
   This is an instance profile role. EC2 instances are automatically created by Elastic Beanstalk. This role ensures the EC2 instance has
   permissions to pull logs, send metrics, and perform AWS operations required by the application.
   These permissions are automatically selected
   ⦁ AWSElasticBeanstalkWebTier
   ⦁ AWSElasticBeanstalkWorkerTier
   ⦁ AWSElasticBeanstalkMulticontainerDocker
   
   ### (II) Role name(EBSservicerole)
   AWS service -- Elastic BeanStalk(Use case) -- Elastic Beanstalk - Environment
   This is the service role for Elastic Beanstalk. Elastic Beanstalk service itself (not the EC2 instance).
   These permissions are automatically selected
   ⦁ AWSElasticBeanstalkEnhancedHealth
   ⦁ AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy

2. Some website templates are downloaded previously. Select any template. Open that template. Select all the files and compress them and send to file with any name.
   Go to EBS
   Create application -- webserver environment -- application name(my-app) -- environment will be selected automatically -- Domain name -- Platform -- managed platform -- tomcat -- select latest version -- upload your code -- version label(version-1) -- Local file -- upload the file -- single instance -- Select the service role -- Select the instance profile role -- Key pair(recommended) -- VPC(default) -- Enable public IP -- Subnets(select 1) -- SSD -- size(12) -- Select default security group -- instance type(t2 small) remove other one -- Monitoring(Leave default settings) -- Create the EBS

⦁	Tomcat is used when thw webpage deployed is HTML or CSS. If it contains java script or aany other select other options.

After creation an instance and S3 bucket is automatically created.

Initially delete the application, then automatically all the environment is deleted that means EC2 instance, Load balancers all will be deleted. Then delete the roles attached to it. Delete the s3 bucket. Delete the policy attached to it, then empty the bucket and now delete the policy.

---

## Task-6:

A lambda function is created and event is scheduled but time is not given.
Create function -- function name -- version python 3.13 -- x86 -- create a new role with basic lambda permissions(1) -- Create Function
Go to IAM Role:
Open the role already created -- Add permissions to the roel -- Add ec2 full access permission to the role
Launch an ec2 instance
Python 3.13 version

```python
import boto3
region = 'us-east-1'
instances = ['i-0602f55d3ab98ae0f']
ec2 = boto3.client('ec2', region_name=region)
def lambda_handler(event, context):
    ec2.stop_instances(InstanceIds=instances)
    print('stopped your instances: ' + str(instances))
```

Go to function created and go to code. Paste the code -- Test event -- Give any name for the event -- Deploy the event(The event is saved) -- Test the event -- The output is shown below -- The changes can be reflected in the ec2 instances
Add trigger -- select Event-Bridge -- Create a new rule -- Give rule name -- rate
⦁	Trigger is added and given a time and for that every time, the event is scheduled.

#### Create a lambda function to create a snapshot of EC2-volume
1. Create an EC2 instance. A volume is automatically created and attached to the EC2 instance.
2. Create a role with EC2FullAccess permissions.
3. Create a lambda function and attach the role to function

```python
import boto3
import datetime
ec2 = boto3.client('ec2')
Replace with your volume ID(s)
VOLUME_IDS = ['vol-xxxxxxxxxxxxxxxxx']  # you can add multiple volumes here
def lambda_handler(event, context):
    for volume_id in VOLUME_IDS:
        timestamp = datetime.datetime.utcnow().strftime('%Y-%m-%d-%H-%M-%S')
        description = f"Snapshot of {volume_id} at {timestamp}"
        try:
            response = ec2.create_snapshot(
                VolumeId=volume_id,
                Description=description,
                TagSpecifications=[
                    {
                        'ResourceType': 'snapshot',
                        'Tags': [
                            {'Key': 'CreatedBy', 'Value': 'Lambda'},
                            {'Key': 'Name', 'Value': f'{volume_id}-snapshot'}
                        ]
                    }
                ]
            )
            print(f"Snapshot created: {response['SnapshotId']}")
        except Exception as e:
            print(f"Error creating snapshot for volume {volume_id}: {str(e)}")
```

4. Test the code and deploy the code. Again test the code.
5. Snapshot is created.
6. Add trigger to it.

Create 2 ec2 instances in 2 availability zones
us-east-1a
us-east-1b
Load balancer -- application load balancer

---

## Task-7:

### Vertical Scaling: 
It means the capacity or storage of the EC2 instance is increased or decreased. Scale up and Scale down process is done manually.

⦁	Scale up - Instance type is increased
⦁	Scale down - Instance type is decreased

1. Launch an EC2 instance. Check whether default page is coming or not.
2. Stop the instance.
3. Instance settings -- change instance type -- t2.medium(Initially it is t2.micro)
4. Now start the instance. The instance type is changed to t2.medium
5. The public IPv4 address is also changed. Copy that IP and check if the default page is opening or not. The default page will open. This is called vertical scaling.
6. Before changing instance type the instance type
7. Again change the instance type to t2.micro and start the instance. Now also the default page works.

### Horizontal Scaling: 
EC2 instances are created based on number of users when needed automatically through the image. To launch another instance we will create an image of the present running instance. All the other instances are launched from the image created only.

Stop the instance.
Select instance -- Actions -- Image and Templates -- Create Image -- Give an image name -- Create Image
An Image is created. Check in AMIs
Now terminate the instance as it is no longer needed.
Create auto scaling
Name - Create launch template -- template name -- My AMIs(Select the image created) -- Instance type(Don't include) -- Key pair created -- security group(launch wizard don't select default) -- create launch template
Launch template - selected the created one --- next -- instance type -- manually add instance types(t2.micro) -- network settings -- select default VPC -- Select atleast 2 availability zones -- next -- attach to new load balances
desired capacity -- 2 instances -- min 2, max 3 -- scaling policy -- target scaling policy -- metric type -- cpu metrics -- change 50 to 20 -- warmup -- 100 -- additional settings default -- next -- create

⦁ If the CPU utilization is 100% then automatically another instance is created. To check this
1. Go to mobaxterm and give the IP address of running instance and connect to it.
2. Go to the root and give the following commands
   apt-get install stress -y --> Stress is installed
   stress -c 2 --> Instance is created
   top --> To check the CPU utilization wheter it is 100% or not
3. Now we can see that an instance is created as the minimun desired capacity is exceeded

---

## Task-8:

The task is about that when you signup or login the data should be stored. So we install mysql and create a table and store the credintials in that table.

1. Launch a EC2 instance. Connect it to the mobaXterm and deploy the sample webpage.
2. Now a sample login page with login and sign up options is there. Deploy that webpage in the place of sample webpage.
3. Install the mysql database server -- ```bash apt install mysql-server```
4. Check the status of the server -- ```bash systemctl status mysql```
5. To secure the serve need to create a password.
6. Go the mysql prompt -- ```bash mysql -u root``` -- This goes to the mysql prompt.
   In my sql prompt:
   ---> Give the password for the mysql server -- ```bash ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'NewPassword';``` (In place of NewPassword give any password and remember)
   ---> Removes any default users and passwords are present -- ```bash FLUSH PRIVILEGES;```
   ---> To see any databases present or not -- ```bash show databases;```
   ---> Create a database -- ```bash CREATE DATABASE user_db;``` -- user_db database is created
   ---> To store anything change to the database created -- ```bash USE user_db;``` -- Changes to the user_db database
   ---> Create a table with id, username and password

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL
);
```

7. Exit from the mysql promt. Now you are in the root location.
8. Now we created a database and also we have the sample login page that is frontend. Now we need backend and that backend and frontend need to be connected.
9. Installs a backend php -- ```bash apt install php libapache2-mod-php php-mysql -y```
10. To see which modules of php are installed -- ```bash ls /etc/apache2/mods-available/ | grep php```
11. To check whether backend is running or not -- ```bash echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php``` -- sample php file is created in /var/www/html
12. Go to the browser and give ```bash ip address/info.php``` -- Sample backend page is opened and thus the backend file is working.
13. Go to the cd /var/www/html and info.php is created there. Remove it from there. We created just to confirm that backend is created or not
14. Download the db.php, login.php, signup.php files to the system.
15. Connect to the SFTP terminal and go to the /var/www/html directory and move the downloaded files to the right side.
16. Go to the SSH terminal and open the db.php file and change the password to the mysql password that is reseted before.
17. Don't change anything in login.php and signup.php files.
18. Open the index.html file and in that go to the script section and change the ip address to our ec2 instance ip address for both login and signup script.
19. Go to the browser and in signup give username as root and password is mysql password(i.e 'mysql') and signup. It will appear as 'Signup successful'.
20. Go to mobaXterm:
    ⦁	 Give the command -- ```bash mysql -u root -p``` -- Enter the password of mysql. Changes to the mysql prompt
    ⦁	 Change database to user_db -- ```bash USE use_db;```
    ⦁	 A table with id, username, password is appeared -- ```bash SELECT * FROM users;```

---

## Task-9:

This task is similar to the above one but an RDS database is created and through that database it is connected.

1. Launch an EC2 instance and allow SSH, HTTP, MYSQL/Aroura Ports in the security group while creating.
2. Create and RDS database and while creating connect to the ec2 instance and select MySQL.
3. Connect to the mobaXterm, update and launch the apache2 sample page.
4. Install the backend php -- apt install php libapache2-mod-php php-mysql -y
5. Now go the directory where index.html page is located (/var/www/html) and replace the sample index.html with the web page containing signup and login options. In that change the ip address.
6. In the same directory also deploy signup.php, login.php, db.php files.
7. In db.php file change 
   (i) host --> Endpoint of RDS
   (ii) dbname --> Database name of rds
   (iii) username --> Name given while creating RDS(admin)
   (iV) Password --> Password given while creating RDS
8. Now go to /home/ubuntu and install mysql server -- apt install mysql-server
9. To go to the SQL prompt -- ```bash mysql -h <dbendpoint> -u username -p``` -- Press Enter and then give the password.
   dbendpoint --> EndPoint of RDS
   username --> Give the username(admin)
10. Create a database -- ```bash CREATE DATABASE database1;``` (Give the same name of database created in RDS)
11. Use that database -- ```bash USE database1;```
12. Create a table users
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL
);
```
13. Open the login, signup webpage in the browser and give some credintials in signup. Then signup successfull will appear.
14. Go to the SQl prompt and give the command -- ```bash SELECT * FROM users;```
15. The signup credintials will appear in the form of table.
16. When logged in, it shows login successful.

⦁	While creating RDS in place of selecting MYSQL if we select MariaDB then download MariaDB server in mobaXterm. It is completely same as MySQL server only. In this way we can select any database.


apt perge
apt install manyadb

Create a lambda function and deploy it
Go to API gateway
End point regional

---

## Task-10:

### ElastiCache: 
Amazon ElastiCache is a web service that improves the performance of applications by allowing you to retrieve information from fast, managed, in-memory caches, instead of relying solely on slower disk-based databases.
⦁	Redis --> Open source cache
⦁	Shard means a small group
⦁	Primary Endpoint(R/w) + Read Replica = Shard
⦁	Multiple shard is called cluster.

### Creating ElastiCache:
Get started
Select redis OSS -- Design your own cache -- Cluster cache
Cluster mode -- Disabled, Cluster info -- Give name
Location -- AWS, Multi AZ -- Disable, Auto-failover -- Disable
Cache settings:
Engine Version - 7.1, Port - 6379, Node type -- t2.micro, Replicas -- 0
Connectivity:
Create new subnet group -- Give name, Select default vpc, select 1 subnet
Advanced Settings: Don't change other settings
Disable backup
Create ElastiCache

Now go the default securuty groups and add redis port number(6379) and also verify if SSH is added.
Launch an Amazon Linux ec2 Instance -- In Network Settings Select default VPC, Select existing security group(default)
Connect to the mobaXterm
Install the required packages:
```bash
yum install gcc
yum install openssl-devel
yum groupinstall "Development Tools" -y
cd ~ --> Goes to the root. Give pwd it should show /pwd
wget [http://download.redis.io/redis-stable.tar.gz](http://download.redis.io/redis-stable.tar.gz) --> Downloads Redis source code
ls --> See if 'redis-stable.tar.gz' is installed or not
tar xvzf redis-stable.tar.gz --> Extract from the zip file
cd redis-stable --> Goes to that directory
make BUILD_TLS=yes
src/redis-cli -c -h ENDPOINT --tls -p 6379 --> In place of ENDPOINT give primary endpoint
(src/redis-cli -c -h master.my-cluster.gk4cos.use1.cache.amazonaws.com --tls -p 6379)
The endpoint will appear. Use the following
set a 'TechnicalHub'  --> OK will come. This means some data is stored in the cache memory
get a --> This gives the data of set a
set b 'AWS' ex 5 --> This means some data is stored in the cache memory but will expire after 5 seconds. Before 5 secongs it will appear.
```

The commands will slightly differ when doing in Ubuntu Server
```bash
⦁	sudo apt update -y
⦁	sudo apt upgrade -y
⦁	sudo apt install -y build-essential tcl libssl-dev wget
⦁	cd ~
⦁	wget [http://download.redis.io/redis-stable.tar.gz](http://download.redis.io/redis-stable.tar.gz)
⦁	tar xvzf redis-stable.tar.gz
⦁	cd redis-stable
⦁	make BUILD_TLS=yes
⦁	./src/redis-cli -c -h <ENDPOINT> --tls -p 6379
The set commands are same as in amazon linux.
```

⦁ IANA - Internet Assigned Numbers Authority

---

1. Launch a ec2 instance(ubuntu). Allow SSH, HTTP ports.
2. Create an elastic ip and associate the elastic ip to the ec2 instance created.
3. Connect to mobaxterm and launch the default page.
4. [https://get.tech/github-student-developer-pack](https://get.tech/github-student-developer-pack) --> Go to this website and search any website with .tech
5. Select any product and add to the cart. Initially it will charge. When you login with github education account it wiil be 0 cost.
6. [https://controlpanel.tech](https://controlpanel.tech) --> Go to this login into it (Select customer)

---

AWS Certificate Manager
certbot
python3-certbot-apache
Letsecrypt - open source ssl
certbot --apache -d vaishubuilding.tech -d [www.vaishubuilding.tech](http://www.vaishubuilding.tech)


ns lookup
[https://ssllabs.com/ssltest/](https://ssllabs.com/ssltest/)
a, a+, b+ -- good ratings
B, c+, c -- bad ratings

---



# DevOps
## Devops Tools:
- Slack
- Jira
- STAND UP
- Microsoft Planner

Container: OS - Linux - Aipine
Common Libraries
- Software --> http, redis, mysql, postgres, nginx
- Docker is a vendor which can be used to create, build, distribute, run, deploy, storage.
- Apart from docker cri-o, container-d, build-ah can also be used but in these vendors, container can only be runned and deployed.


### Installing Docker:

1. Launch an instance of Ubuntu server and then connect to mobaXterm.
2. Update the instance -- ```bash apt-get update```
3. To install docker --> ```bash apt install docker.io```
4. To check the version of the docker --> ```bash docker --version```
5. To see the list of images in docker --> ```bash docker images```
6. To see if any container is running --> ```bash docker ps```
   ⦁	Running environment of image is called container.
   To download container images they are available in docker hub. We can choose any of the image.
7. To download containers --> ```bash docker pull image_name``` (Ex: docker pull nginx) --> This downloads the latest version of the image
   ```bash docker pull nginx:9.5.0``` --> This downloads the mentioned version
8. To run the container image in the backend --> ```bash docker run -d nginx```
   ```bash docker run nginx``` --> The image will run in the foreground. It may disturb our work.
9. To stop the running container image --> ```bash docker stop container_id```
10. To see how many containers stopped --> ```bash docker ps -a```
11. To start the stopped container --> ```bash docker start name```(docker start nginx)
12. If the image is not pulled and even if we try to run that image then it takes that image from the library --> ```bash docker run redis```
13. To remove the stopped container --> ```bash docker rm container_id```
14. To remove the image --> ```bash docker rmi image_name```
15. To go the bash terminal --> ```bash docker exec -it container_id /bin/bash```
16. To change the name of the of the container --> ```bash docker run -d --name [mention the customized name] [exiting image name]```
    ⦁	Port numbers are very important because containers communicate through port numbers internally.
17. To change the  port number of the container --> ```bash docker run -d -p[port number we want]:[default port number] imagename```
    Ex: (docker run -d -p6001:6379 imagename)
18. docker run -d --name my-app -p8980:80 httpd

Following is the command to copy website files and directories to docker container

```bash docker cp source/file/or/directory containerid:/path/to/destination```

---

### Deploying a sample website into docker:
1. Install the docker.
2. Pull any image like nginx, httpd or apache as websites can be deployed in it only. (```bash docker pull nginx```)
3. Run the image (```bash docker run -d nginx```)
4. Change the port number from 80 to 8080 and also name.
   This changes the port number and name from nginx to my-nginx --> ```bash docker run -d -p 8080:80 --name my-nginx nginx```
5. Pull the postgres image(```bash docker pull postgres```). Fot httpd and apache it is not needed but for nginx postgres need to be pulled.
6. Now we can see two containers of nginx. One is default and other is the container with the name we customized.
7. If we give the ip address along with port number we we'll get the sample website.(Ex: [http://34.239.152.47:8080](http://34.239.152.47:8080))
8. Deploy that website into docker hub repository. To deploy follow the below commands
```bash
   ⦁	docker commit <container_id> <new_image_name>:<tag>
   ⦁	docker tag <image_id> <docker_hub_username>/<repository_name>:<tag>
   ⦁	docker login -u <docker_hub_username>
   ⦁	docker push <docker_hub_username>/<repository_name>:<tag>
```
Deploying the website in the above process is time taking. So below is another process.

1. Launch ubuntu instance.
2. Apdate and install the apache2
3. Create Dockerfile -- vim Dockerfile
   In the vim file
   FROM ubuntu:22.04                   --> Installs the ubunutu OS
   RUN apt-get update -y               --> Update the ubuntu
   RUN apt-get install apache2 -y      --> Install and run the apache2
   COPY . /var/www/html        	       --> . means Copy from current location
   EXPOSE 80		    -->
   CMD ["apachectl", "-D", "FOREGROUND"]

#### Q. Can I use systemctl instead of CMD?
No because system-d service is not present in the ubuntu as it is a light weight service

After writing the code in vim file save it and run the following commands.
```bash
docker build -t web-app .
docker images
docker run -d p80:80 web-app
```
Give the ip address and check if the web-page is opening.

We can directly install ubuntu and apache2 with single command.
```bash
FROM ubuntu/apache2:latest
COPY . /var/www/html
EXPOSE 80
```
The following commands for downloading ngnix
```bash
FROM nginx:latest
COPY ./usr/share/nginx/html
EXPOSE 80
CMD["nginx", "-g", "daemon off;"]
```

---

### Demo Project:
node app runs in Virtual Machine
mongodb runs in container
node-app and mongodb are connected.
Application in virtual machine and database in container.

environment variables of mongo image:
⦁ MONGO_INITDB_ROOT_USERNAME: root
⦁ MONGO_INITDB_ROOT_PASSWORD: example

1. Launch an instance.
2. Apdate the VM and Install the docker
3. Search for mongo image in docker hub
4. In the bottom, copy the environment variables in the yaml file
5. To see the default networks  --> ```bash docker network ls```
6. mongo-net network is created  --> ```bash docker network create mongo-net```
7. ```bash docker run -d --name mongodb --network mongo-net -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=example mongo:latest```
   The above command can be split to understand more better
   --name mongodb
   --network mongo-network
   -p 27017:27017
   -e MONGO_INITDB_ROOT_USERNAME: root
   -e MONGO_INITDB_ROOT_PASSWORD: example
   mongo:latest\

8. Enable the port number of mongodb(27017) in security groups of instance.
9. ```bash docker run -d --name mongo-express --network mongo-net -p8081:8081 -e ME_CONFIG_MONGODB_ADMINUSERNAME=root -e ME_CONFIG_MONGODB_ADMINPASSWORD=example -e ME_CONFIG_MONGODB_SERVER=mongodb -e ME_CONFIG_MONGODB_PORT=27017 -e ME_CONFIG_BASICAUTH_USERNAME=admin -e ME_CONFIG_BASICAUTH_PASSWORD=admin123 mongo-express:latest```
   ⦁	ME_CONFIG_MONGODB_ADMINUSERNAME  &&  ME_CONFIG_MONGODB_ADMINPASSWORD --> These two are to connect to mongodb
   ⦁	ME_CONFIG_BASICAUTH_USERNAME  &&  ME_CONFIG_BASICAUTH_PASSWORD  --> These two are for mongo-express UI
   ⦁	ME_CONFIG_MONGODB_SERVER:mongodb  --> Mentions for which server we are connecting
   ⦁	ME_CONFIG_MONGODB_PORT:27017  --> Port number of the server we are connecting
   ⦁	8081 --> port number of mongo-express
   ⦁	27017 --> port number of mongo

10. Enable the port number mongo-express(8081) in security groups
11. Give the ip address of the instance with port number 8081 and open(http:18.207.195.188:8081). A mongodb database page will appear.

After the above process
Create a directory -- ```bash mkdir node-app```
Go into that directory -- ```bash cd node-app```
Run the below commands (Actually node commands are not working but still try them)
```bash
apt install npm
npm init -y
apt install node --> Not Working
node init -y  --> Not Working
npm install express mongoose body-parser cors
```

In the node-app directory create public directory (mkdir public) and in that copy index.html file. Come out of that public directory
and create server.js file in the node-app directory. (The index.html, server.js files are given)
node-app --> public directory -> index.html
         --> server.js
Run the below command
```bash node server.js``` --> Runs the server.js in 3000 port number
Give the ip address along with port number 3000 (http:18.207.195.188:3000). Index.html file will be opened. Save to mongoDB.
Open the mongodb database page. You can see the database with name user-account. Open it and will be able to see the details.

Now commit the mongodb into docker hub and AWS ECR
Commit mongodb -> tag -> push to the docker hub
                         push to aws ECR

---

### Elastic Container Registry(ECR)
⦁	Create ECR and give the name and create(thub-app)
⦁	Select the name after created and select push commands
Install AWS CLI in mobaXterm to push the container into the ECR. Below are the commands to install AWS CLI
```bash
sudo apt update
sudo apt install -y unzip curl
curl "[https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip](https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip)" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
In the root account Create access key and configure with aws cli using command (```bash aws configure```)


