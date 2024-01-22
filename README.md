# AWS 3-Tier Architecture Guide 🌐

This repository provides a comprehensive guide for setting up a 3-tier AWS architecture. The architecture comprises a VPC, multiple subnets, EC2 instances, a bastion host, RDS database, and an Application Load Balancer.

## Prerequisites 📝
- An AWS account with IAM user access
- Familiarity with the AWS Console
- Familiarity with VPC network structures, EC2 instances, Auto scaling groups, and security groups.
-  Familiarity with Linux commands, scripting, and SSH.
-  Access to a command line tool.

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
    - NAT GatewayL `None`
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
- Navigate tp `EC2 -> Auto Scaling Group -> Create Auto Scaling Group`
- Name group `webserver-asg` and choose the Launch Template you just created
- Choose the webapp-vpc and select the 2 public subject under the `Availabilty Zones and subnets`

![3tier-step2-c](https://github.com/Btech31488/aws3tierproject/assets/104148696/393971eb-41e9-41d7-ac4a-9122f5ca32c8)

- On the next page, choose the option `Attach to a new load balancer`, name the load balancer `webserver-alb`, choose the `Internet-facing` option, make sure the 2 public subnets are selcted under the `Availability Zones and subnets`

![3tier-step2-d](https://github.com/Btech31488/aws3tierproject/assets/104148696/2f7f1284-55ce-4622-8214-b8215abe92a6)

- Under `Listeners and routing` Choose `Create a target group` option in the 1Default routing (forward to)` drop down and name the target group `webserver-tg`

![3tier-step2-d-2](https://github.com/Btech31488/aws3tierproject/assets/104148696/08c0fc87-bbfb-47cf-8d6f-e4792ecd0d5d)

- On the next page it will have you create a scaling policy. Choose the Desire capacity to `2`, Min desired to `2`, and Max to `5`

![3tier-step2-e](https://github.com/Btech31488/aws3tierproject/assets/104148696/022fa02f-900e-4466-a330-b075e23c359d)

- Under the "Automatic Scaling" session choose `Target tracking scaling policy`, under "Metric type" choose `Average CPU utilization`. Set the "Target value" to `50` and  "Instance warmup" to `300` (This will set to where if the instance CPU is use is at 50% it will scale up and when it's below, scale back down. Warm-up time for the new instance is 300 secs)

![3tier-step2-e-2](https://github.com/Btech31488/aws3tierproject/assets/104148696/0ea525f8-1f5e-4b09-921c-1f54675cc741)

#### 3. **Testing Connectivty**
- Once the auto scaling group and load balancer is provisioned, head to `EC2 -> Load Balancers -> The load balancer created` and copy the DNS name.
- Paste it in your browser's address bar to test to see if you can access the sample website.
 
![3tier-step2-e-3](https://github.com/Btech31488/aws3tierproject/assets/104148696/3965c697-3be5-47a3-9089-52bbf61c37a5)

- SSH in from yout local terminal to confirm connectivity and access

![3tier-step2-e-4](https://github.com/Btech31488/aws3tierproject/assets/104148696/6bdd4332-70e8-49cb-b24f-6451a1ef15fd)

### Tier 2: Web tier (Backend)

![3TierVPCArchTier2](https://github.com/Btech31488/aws3tierproject/assets/104148696/f34ebfaf-5f04-4e23-a076-d24f54cb85da)


### 7. **Load Balancer & Target Group**
- Navigate to `EC2 Dashboard -> Load Balancers -> Create Load Balancer`.
    - Choose `Application Load Balancer`.
    - Create a new target group named `MyTG` and register your web servers.

### 8. **RDS Database Setup**
- Go to `Services -> RDS -> Create database`.
    - Ensure it's within `My3TierVPC`.
    - Create a `Subnet Group` comprising all your `PrivateDBSubnetX`.
    - Security Group: Allow port `3306` only from `WebSG`.

### 9. **Web Server Database Configuration**
- SSH into web servers via the bastion host.
    ```bash
    cd /var/www/html/phpMyAdmin/
    ```
    ```bash
    #change the name of the php file
    mv config.sample.inc.php config.inc.php
    ```
    ```bash
    vim config.inc.php
    ```
    - Change `$cfg['Servers'][$i]['host'] = 'localhost';` to `$cfg['Servers'][$i]['host'] = '<DATABASE_ENDPOINT>';`
    - Replace `<DATABASE_ENDPOINT>` with your actual database endpoint address.
    
- Navigate to the target group (MyTG) in AWS console and enable stickiness.

### 10. **Testing**
- Access web servers via the ALB DNS and verify the connection with the database (http://my.public.dns.amazonaws.com/phpMyAdmin)

## Note ⚠️
This is a basic guide, primarily for educational purposes. In a production environment, additional configurations related to security, monitoring, scaling, backups, etc., are vital.
