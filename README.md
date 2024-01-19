# AWS 3-Tier Architecture Guide 🌐

This repository provides a comprehensive guide for setting up a 3-tier AWS architecture. The architecture comprises a VPC, multiple subnets, EC2 instances, a bastion host, RDS database, and an Application Load Balancer.

## Prerequisites 📝
- Sign in to your AWS Management Console.

## Steps to Setup 🛠

### 1. **VPC Creation**
- Navigate to `Services -> VPC -> Your VPCs -> Create VPC`.
    - Name: `My3TierVPC`
    - CIDR Block: `10.0.0.0/16`
    - Click on `Create`.

### 2. **Subnet Creation**
- **Public Subnets**:
    - Before you create your subnets, make sure you filter your VPC to the newly created `My3TierVPC`
    - Repeat for 3 subnets: `10.0.1.0/24`, `10.0.2.0/24`, `10.0.3.0/24`:
        - Go to `Subnets -> Create subnet`.
            - Name: `PublicSubnetX` (Replace X with 1, 2, or 3)
            - VPC: `My3TierVPC`
            - Availability Zone: Different for each subnet (for me I used AZ a, b, c).
            - CIDR Block: Use the corresponding CIDR.
            - Click `Create`.
    
- **Private Web Subnets**:
    - Repeat as above for 3 subnets using the CIDRs `10.0.4.0/24`, `10.0.5.0/24`, `10.0.6.0/24`, naming them `PrivateWebSubnetX`.
    
- **Private DB Subnets**:
    - Create 3 more subnets using the CIDRs `10.0.7.0/24`, `10.0.8.0/24`, `10.0.9.0/24`, naming them `PrivateDBSubnetX`.

### 3. **Internet Gateway (IGW) Setup**
- Go to `Internet Gateways -> Create internet gateway`.
    - Name: `MyIGW`
    - Attach it to `My3TierVPC`.

### 4. **NAT Gateway Configuration**
- Navigate to `Elastic IPs -> Allocate new address -> Allocate` and note the IP.
- Proceed to `NAT Gateways -> Create NAT gateway`.
    - Choose a public subnet and the previously allocated Elastic IP.
    - Click on `Create`.

### 5. **Route Table Configuration**
- **Public Route Table**:
    - Go to `Route Tables -> Create route table`.
        - Name: `PublicRT`
        - VPC: `My3TierVPC`
    - Edit routes to add `0.0.0.0/0` targeted at `MyIGW` and associate with public subnets.

- **Private Web Route Table**:
    - Create a table named `PrivateWebRT`, add a route targeted at NAT gateway, and associate with private web subnets.

- **Private DB Route Table**:
    - Create `PrivateDBRT` and associate with private DB subnets.

### 6. **Bastion, Web Servers, and LAMP Installation**
- **Bastion Host**:
    - Go to `Services -> EC2 -> Launch Instance`.
        - Choose `Amazon Linux AMI`.
        - Configure instance details under `My3TierVPC` in a public subnet.
        - Security Group: Allow SSH from your IP.
        - Launch and select/create a key pair.

- **Web Servers**:
    - Launch 2 instances in private web subnets, allowing SSH only from `BastionSG`.

- **LAMP Installation**:
    ```bash
    sudo dnf update -y
    ```
    ```bash
    sudo dnf install -y httpd wget php-fpm php-mysqli php-json php php-devel
    ```
    ```bash
    sudo systemctl start httpd
    ```
    ```bash
    sudo systemctl enable httpd
    ```
    ```bash
    sudo usermod -a -G apache ec2-user
    ```
    ```bash
    #Exit and ssh back into the Webserver
    exit
    ```
    ```bash
    #Change the group ownership of /var/www and its contents to the apache group.
    sudo chown -R ec2-user:apache /var/www
    ```
    ```bash
    #Change the group ownership of /var/www and its contents to the apache group.
    sudo chown -R ec2-user:apache /var/www
    ```
    ```bash
    #To add group write permissions and to set the group ID on future subdirectories, change the directory permissions of /var/www and its subdirectories.
    sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
    ```
    ```bash
    #To add group write permissions, recursively change the file permissions of /var/www and its subdirectories
    find /var/www -type f -exec sudo chmod 0664 {} \;
    ```
    ```bash
    #Install the required dependencies
    sudo dnf install php-mbstring php-xml -y
    ```
    ```bash
    #Restart Apache
    sudo systemctl restart httpd
    ```
    ```bash
    #Restart php-fpm
    sudo systemctl restart php-fpm
    ```
    ```bash
    #Navigate to the Apache document root at /var/www/html
    cd /var/www/html
    ```
    ```bash
    #Download the file directly to your instance, copy the link and paste it into a wget command
    wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
    ```
    ```bash
    #Create a phpMyAdmin folder and extract the package into it with the following command
    mkdir phpMyAdmin && tar -xvzf phpMyAdmin-latest-all-languages.tar.gz -C phpMyAdmin --strip-components 1
    ```
    ```bash
    #Delete the phpMyAdmin-latest-all-languages.tar.gz tarball
    rm phpMyAdmin-latest-all-languages.tar.gz
    ```
    
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
