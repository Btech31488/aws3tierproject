# AWS 3-Tier Architecture Guide 🌐

This repository provides a comprehensive guide for setting up a 3-tier AWS architecture. The architecture comprises a VPC, multiple subnets, EC2 instances, a bastion host, RDS database, Application Load Balancer and Auto Scaling.

## Prerequisites 📝
- An AWS account with IAM user access
- Familiarity with the AWS Console
- Familiarity with VPC network structures, EC2 instances, Auto scaling groups, and security groups.
- Familiarity with Linux commands, scripting, and SSH.
- Access to a command line tool.

## Why 3 Tier architecture?

Why opt for a 3-tier architecture? This architectural approach effectively tackles the mentioned challenges by enhancing scalability, availability, and security. It achieves this by distributing the application across multiple Availability Zones and dividing it into three distinct layers, each with its own unique role. In the event of an Availability Zone failure, the application seamlessly adjusts by automatically allocating resources to another zone without disrupting the other tiers. Additionally, each tier maintains its own security group, permitting only the necessary inbound and outbound traffic for specific tasks. 

## Steps to Setup 🛠

### Network Architecture

![3TierVPCArch](https://github.com/Btech31488/aws3tierproject/assets/104148696/73c3db3e-08b0-4ba6-a27b-ce903a87495c)


#### 1. **VPC Creation**
- Navigate to `Services -> VPC -> Your VPCs -> Create VPC`.
    - Choose `VPC and more` option
    - Name: `webapp-vpc`
    - CIDR Block: `10.0.0.0/16`
    - Number of Availabilty Zones, (AZs): `2`
    - Number of public subnets: `2`
    - NAT Gateway `None`
    - VPC endpoints: `None`
    - Leave everything else as default and click `Create`.
    - This will automatically create you VPC, public and private subnets under your choosen CIDR, assign each subnet with a CIDR block, create the routing tables and associate them with their respective subnets, and create an Internet Gateway and attach it to the VPC.
    
![3tier-step1-a](https://github.com/Btech31488/aws3tierproject/assets/104148696/1b6ed7f2-c3ae-4034-aa12-40e641caf969)  ![3tier-step1-b](https://github.com/Btech31488/aws3tierproject/assets/104148696/fdb931fd-0faf-4604-8934-86fe19c01aee)

![3tier-step1-c](https://github.com/Btech31488/aws3tierproject/assets/104148696/b88b03e5-dfb9-4e3c-836d-27aa66ea8ae0)

#### 2. **Enable Auto-assign Public IPv4 to the Public Subnets**
- Navigate to `VPC -> Subnets ->` select a public subnet `webapp-vpc-subnet-public1-us-east-1a` then proceed to `Actions` located at the top right and select `Edit Subnet`
- Check the box labeled `Enable auto-assign public IPv4 address` then click `Save`
- Repeat this step for the subnet `webapp-vpc-subnet-public2-us-east-1b`

![3tier-step1-d](https://github.com/Btech31488/aws3tierproject/assets/104148696/cb1f87b8-6cb5-4a27-84ae-466cc9fc6d34)


#### 3. **Public Route Table Configuration**
- Go to `Route Tables` select the public route table name `webapp-vpc-rtb-public` the navigate to `Actions` and select `Set main route table`.

![3tier-step1-e](https://github.com/Btech31488/aws3tierproject/assets/104148696/4fd5dda7-d882-4d8e-afb3-cf18cce7d2e0)

 
#### 4. **NAT Gateway Configuration**
- Navigate to `Elastic IPs -> Allocate new address -> Allocate` and note the IP.
- Proceed to `NAT Gateways -> Create NAT gateway`.
    - Choose a public subnet and the previously allocated Elastic IP.
    - Click on `Create`.

![3tier-step1-f](https://github.com/Btech31488/aws3tierproject/assets/104148696/b50d2b8b-60a7-4020-9457-ba036041de3e)

#### 5. **Private Route Table Configuration**
- Go to `Route Tables` select the public route table name `webapp-vpc-rtb-private` the navigate to `Actions` and select `Edit subnets associations`.
- Select all the private subnets and save the changes.

![3tier-step1-g](https://github.com/Btech31488/aws3tierproject/assets/104148696/6c2c6875-35ec-4d1a-b8b2-36097e4ef49f)

- Once done delete the other 3 private routing table since we only need 1. If done correctly there should be only 2 routing tables. One with 2 associated subnets and the other with 4 associated subnets.
- Finally, add a route in your private routing table to the NAT Gateway navigating to `Actions` and select `Edit routes` then add a route with Destination `0.0.0.0` and Target the NAT Gateway you created.

![3tier-step1-h](https://github.com/Btech31488/aws3tierproject/assets/104148696/ccc1f139-2f30-4ff2-8d4c-f073380a561d)

### Tier 1: Web tier (Frontend)

![3TierVPCArchTier1](https://github.com/Btech31488/aws3tierproject/assets/104148696/54584c96-91e4-4ac5-92ea-d1409386392e)


#### 1. **Create the Web Server template**
- **Web Server**:
    - Go to `Services -> EC2 -> Launch Template`.
        - Choose `Amazon Linux AMI 2023` type `t2.micro` create a key pair.
        - We not going to specidy the VPC
        - Security Group: Name the security group `webserver-sg` and allow SSH from your IP, HTTP and HTTPS from `0.0.0.0`.
![3tier-step2-a](https://github.com/Btech31488/aws3tierproject/assets/104148696/12380ba9-d95a-4c38-97f6-52d13f3dcf83)  ![3tier-step2-b](https://github.com/Btech31488/aws3tierproject/assets/104148696/d3c7984f-9c3b-4059-b3fc-750c73dfaca1)

        - Navigate to `User Data` under `Additional Details and add the following script on the text box to install the Apache server and the sample webpage.
          ```bash
          #!/bin/bash

          #install apache
          sudo yum -y install httpd

          #enable and start apache
          sudo systemctl enable httpd
          sudo systemctl start httpd

          sudo echo '<!DOCTYPE html>

          <html lang="en">
	          <head>
		          <meta charset="utf-8" />
		          <meta name="viewport" content="width=device-width, initial-scale=1" />

		          <title>A Basic HTML5 Template</title>

		          <link rel="preconnect" href="https://fonts.googleapis.com" />
		          <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
		          <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700;800&display=swap" rel="stylesheet" />

		          <link rel="stylesheet" href="css/styles.css?v=1.0" />
	          </head>

	          <body>
		        <div class="wrapper">
			        <div class="container">
				        <h1>Welcome! An Apache web server has been started successfully.</h1>
				        <p>Replace this with your own index.html file in /var/www/html.</p>
			        </div>
		        </div>
	          </body>
            </html>

            <style>
	            body {
		                background-color: #34333d;
		                display: flex;
		                align-items: center;
		                justify-content: center;
		                font-family: Inter;
		                padding-top: 128px;
	             }

	                .container {
		                        box-sizing: border-box;
		                        width: 741px;
		                        height: 449px;
		                        display: flex;
		                        flex-direction: column;
		                        justify-content: center;
		                        align-items: flex-start;
		                        padding: 48px 48px 48px 48px;
		                        box-shadow: 0px 1px 32px 11px rgba(38, 37, 44, 0.49);
		                        background-color: #5d5b6b;
		                        overflow: hidden;
		                        align-content: flex-start;
		                        flex-wrap: nowrap;
            		            gap: 24;
		                        border-radius: 24px;
	             }

	             .container h1 {
		                        flex-shrink: 0;
		                        width: 100%;
		                        height: auto; /* 144px */
		                        position: relative;
		                        color: #ffffff;
		                        line-height: 1.2;
		                        font-size: 40px;
	             }
	            .container p {
		                        position: relative;
		                        color: #ffffff;
		                        line-height: 1.2;
		                        font-size: 18px;
	            }
          </style>
          ' > /var/www/html/index.html
          ```

#### 2. **Create an Auto Scaling Group**
- Navigate to `EC2 -> Auto Scaling Group -> Create Auto Scaling Group`
- Name group `webserver-asg` and choose the Launch Template you just created
- Choose the webapp-vpc and select the 2 public subnet under the `Availabilty Zones and subnets`

![3tier-step2-c](https://github.com/Btech31488/aws3tierproject/assets/104148696/393971eb-41e9-41d7-ac4a-9122f5ca32c8)

- On the next page, choose the option `Attach to a new load balancer`, name the load balancer `webserver-alb`, choose the `Internet-facing` option, make sure the 2 public subnets are selected under the `Availability Zones and subnets`

![3tier-step2-d](https://github.com/Btech31488/aws3tierproject/assets/104148696/2f7f1284-55ce-4622-8214-b8215abe92a6)

- Under `Listeners and routing` Choose `Create a target group` option in the Default routing (forward to)` drop down and name the target group `webserver-tg`

![3tier-step2-d-2](https://github.com/Btech31488/aws3tierproject/assets/104148696/08c0fc87-bbfb-47cf-8d6f-e4792ecd0d5d)

- On the next page it will have you create a scaling policy. Choose the Desire capacity to `2`, Min desired to `2`, and Max to `5`

![3tier-step2-e](https://github.com/Btech31488/aws3tierproject/assets/104148696/022fa02f-900e-4466-a330-b075e23c359d)

- Under the "Automatic Scaling" session choose `Target tracking scaling policy`, under "Metric type" choose `Average CPU utilization`. Set the "Target value" to `50` and  "Instance warmup" to `300` (This will set to where if the instance CPU is use is at 50% it will scale up and when it's below, scale back down. Warm-up time for the new instance is 300 secs)

![3tier-step2-e-2](https://github.com/Btech31488/aws3tierproject/assets/104148696/0ea525f8-1f5e-4b09-921c-1f54675cc741)

#### 3. **Testing Connectivty to Web Server**
- Once the auto scaling group and load balancer is provisioned, head to `EC2 -> Load Balancers -> The load balancer created` and copy the DNS name.
- Paste it in your browser's address bar to test to see if you can access the sample website.
 
![3tier-step2-e-3](https://github.com/Btech31488/aws3tierproject/assets/104148696/3965c697-3be5-47a3-9089-52bbf61c37a5)

- SSH in from yout local terminal to confirm connectivity and access

![3tier-step2-e-4](https://github.com/Btech31488/aws3tierproject/assets/104148696/6bdd4332-70e8-49cb-b24f-6451a1ef15fd)

### Tier 2: App tier (Backend)

![3TierVPCArchTier2](https://github.com/Btech31488/aws3tierproject/assets/104148696/f34ebfaf-5f04-4e23-a076-d24f54cb85da)

#### 1. **Create the App Server template**
- **Web Server**:
    - Go to `Services -> EC2 -> Launch Template`.
        - Choose `Amazon Linux AMI 2023` type `t2.micro` use the same keypair created from previous step.
        - We not going to specify the VPC
        - Security Group: Name the security group `appServer-sg` and allow SSH and IMCP from the `webserver-sg`

![3tier-step3-a](https://github.com/Btech31488/aws3tierproject/assets/104148696/13413464-6fd7-4d5e-a1f8-10fb97c46f0d)  ![3tier-step3-a2](https://github.com/Btech31488/aws3tierproject/assets/104148696/4e4a3c24-3c24-4c63-b15f-81b6d9d32400)

- Navigate to `User Data` under `Additional Details and add the following script on the text box to install the MariaDB.
  ```bash
  #!/bin/bash
  sudo dnf install mariadb105-server -y
  sudo systemctl start mariadb
  sudo systemctl enable mariadb
  ```
#### 2. **Create the App server Autoscaling Group**
- Navigate tp `EC2 -> Auto Scaling Group -> Create Auto Scaling Group`
- We not going to specify the VPC
- Name group `appserver-asg` and choose the Launch Template you just created
- Choose the webapp-vpc and select the 2 private subnet (private1 and private2) under the `Availabilty Zones and subnets`

![3tier-step3-a3](https://github.com/Btech31488/aws3tierproject/assets/104148696/51543781-c47d-4429-aff4-348ad50fb447)

- On the next page, choose the option `Attach to a new load balancer`, name the load balancer `webserver-alb`, choose the `Internal` option, make sure the 2 private subnets are selected under the `Availability Zones and subnets`

![3tier-step3-a4](https://github.com/Btech31488/aws3tierproject/assets/104148696/7997d728-9536-42f0-b61b-b9feb96f390f)

- Under `Listeners and routing` Choose `Create a target group` option in the Default routing (forward to)` drop down and name the target group `appserver-tg`

![3tier-step3-a5](https://github.com/Btech31488/aws3tierproject/assets/104148696/a827719d-5682-4f85-a6f4-893a8fc3bd51)

- On the next page it will have you create a scaling policy. Choose the Desire capacity to `2`, Min desired to `2`, and Max to `5`

![3tier-step3-a6](https://github.com/Btech31488/aws3tierproject/assets/104148696/d73b431f-3fb2-4a74-bb21-66fdddecb8b6)

- Under the "Automatic Scaling" session choose `Target tracking scaling policy`, under "Metric type" choose `Average CPU utilization`. Set the "Target value" to `50` and  "Instance warmup" to `300`

![3tier-step3-a7](https://github.com/Btech31488/aws3tierproject/assets/104148696/e3106f18-6ba4-43b9-b1c4-ab29d8b0d002)

#### 3. **Testing Connectivty to App Server from Web Server**
- SSH into the webserver EC2 and ping the the private IP address of one of the app servers
```bash
ssh -i "whatever_you_name_key.pem" ec2-user@webserverdnsaddress.com
```
- Once logged in to the webserver, ping the appserver IP address. 

 ![3tier-step3-a8](https://github.com/Btech31488/aws3tierproject/assets/104148696/8c2cced3-3efb-4d53-be1a-f903f54f04ea)
 
#### 4. **Create a Bastion Host**
- Go to `Services -> EC2 -> Launch Instance`.
- Choose the VPC that was created and choose the first public subnet in us-east-1a
- Name group `bastion-sg` and allow only ssh from your IP address

![3tier-step3-b](https://github.com/Btech31488/aws3tierproject/assets/104148696/10b00aab-b703-419c-b1f6-3cc83e8f4171)  ![3tier-step3-b2](https://github.com/Btech31488/aws3tierproject/assets/104148696/9b81b6a2-1c5e-4711-bf8f-3831ff8b8ef6)

#### 5. **Create a Bastion Host**
- Navigate to `EC2 -> Security Groups -> and select the appServer-sg group` go to `Action` and select `Edit inbound rules`
- Allow SSH only from the `bastion-sg` security group

![3tier-step3-b3](https://github.com/Btech31488/aws3tierproject/assets/104148696/3fe4a3d3-e4be-44c9-8d93-f4067c1188f5)


### 6. **Test Connection to the Bastion Server**
- In your local terminal SSH into the bastion host
```bash
ssh -i "whatever_you_name_key.pem" ec2-user@webserverdnsaddress.com
```
- Then within the bastion server, ssh into the app server using the IP address. (Make sure you transfer the public key to the bastion host)

![image](https://github.com/Btech31488/aws3tierproject/assets/104148696/54071ab5-089b-4a96-ac87-dbf83b5807da)

### Tier 3: Database Tier (Data storage & retrieval)

![3TierVPCArchTier3](https://github.com/Btech31488/aws3tierproject/assets/104148696/030239dd-d410-4754-a4a0-8121cd00133a)

### 1. **Create a database security group**
- Navigate to `EC2 -> Security Groups` and create a new security group

![3tier-step4-a](https://github.com/Btech31488/aws3tierproject/assets/104148696/97f53de2-8612-41f6-8453-c11a81befd3b)


- Add the inbound AND outbond rule that allow MySQL 3306 from the `appserver-sg`

![3tier-step4-a2](https://github.com/Btech31488/aws3tierproject/assets/104148696/2f1eff91-fe5f-4bde-8623-87cbc2d746f1)

- Navigate security group `appserver-sg` and add the inbound AND outbond rule that allow MySQL 3306 from the `db-sg` group you just created

![3tier-step4-a3](https://github.com/Btech31488/aws3tierproject/assets/104148696/ccec89e1-21b9-4ab9-8beb-b094b2a7581a)  ![3tier-step4-a3-2](https://github.com/Btech31488/aws3tierproject/assets/104148696/30c25068-071c-45c1-8979-ff391e06477a)

### 2. **Create DB Subnet Group**
- Navigate to `RDS -> Subnet groups -> Create DB subnet group`
- Make sure you choose the VPC that was created for this project

![3tier-step4-b](https://github.com/Btech31488/aws3tierproject/assets/104148696/c8f54aa0-ba03-4b37-8dec-9b5e5aae3e91)

- Under "add subnets" select the AZs `us-east-1 and us-east-2` and the last 2 private subnets `subnet-private3 and -subnet-private4`
  - NOTE: The drop down dont provide the subnet names, so navigate back to the main subnet dashboard to retrieve the current subnets

![3tier-step4-b2-1](https://github.com/Btech31488/aws3tierproject/assets/104148696/d3b18a82-dab1-4190-abe2-03c6d62dbb7a)
  ![3tier-step4-b2](https://github.com/Btech31488/aws3tierproject/assets/104148696/33028100-0134-45c6-824f-c670cef0bc2e)

### 3. **Create RDS Database**
- Navigate back to the RDS Dashboard and select `Create Database`
- Under "Engine Options" select `MySQL`

![3tier-step4-c](https://github.com/Btech31488/aws3tierproject/assets/104148696/b21cc446-4a14-40d5-86d8-bb8f899ed1c3)

- Under the "template" we will select the `Free Tier` option

![3tier-step4-c2](https://github.com/Btech31488/aws3tierproject/assets/104148696/9bcacc8f-9bd7-4368-852a-74959a2d4c16)

- Under "Settings" we will name out DB identifier `webApp-db`, make the master username `admin`, create and confirm the password

![3tier-step4-c3](https://github.com/Btech31488/aws3tierproject/assets/104148696/3f126469-8e6b-4727-9688-cad752edd5bb)


- Under "Connectivity", select `Don't connect to an EC2 computer resource` and make sure you choose the VPC we created for this project

![3tier-step4-c4](https://github.com/Btech31488/aws3tierproject/assets/104148696/f1348fbd-6c9b-4b4d-a565-efd420f51dd7)

- Under the DB subnet group make sure you select the one you just created in the previous step. Also son't allow public access.

![3tier-step4-c5](https://github.com/Btech31488/aws3tierproject/assets/104148696/71a722d6-449e-4a90-994d-a422d321e375)

- For the security group select `Choose existing` and select the `db-sg` that we created. For the AZ select `us-east-a`

![3tier-step4-c6](https://github.com/Btech31488/aws3tierproject/assets/104148696/d60cf178-0c08-4308-bfe8-baca903efd34)

- Under "Additional configuration" repeat the name of the datatbase you created from the first step
- Leave everything else as default, please not RDS will take time to provision.

![3tier-step4-c7](https://github.com/Btech31488/aws3tierproject/assets/104148696/4f9f148b-bcd9-4a1e-8e59-d5960859c342)

### 4. **Connect to the Database**
- Once the RDS is provisioned, retrieve the endpoint address from the dashboard 

![3tier-step4-c8](https://github.com/Btech31488/aws3tierproject/assets/104148696/21a3b971-3b98-40eb-ab67-997d252d2e31)

- If you haven't yet, ssh in to the bastion host, then ssh in to the app server
- In the app server run the command below to connect to the database

```bash
mysql -h YOUR_DB_ENDPOINT -P 3306 -u YOUR_DB_USERNAME -p
```

- When prompted, enter the password you created for the DB

![3tier-step4-c9](https://github.com/Btech31488/aws3tierproject/assets/104148696/da7367f8-0ae6-4896-934f-2f2a25f5b805)

- Great!!!! you have successfully access the DB server

This was quite a journey but I love it when a plan come together. We've successfully created a highly avaliable and scalable application architecture for our webapp!

## Note ⚠️
This is a basic guide, primarily for educational purposes. In a production environment, additional configurations related to security, monitoring, scaling, backups, etc., are vital. Make sure you delete all resources and release the Elastic IP to avoid unneccary charges.
