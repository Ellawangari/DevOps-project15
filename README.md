# Starting Off  AWS Cloud Project


## Pretasks:
- Configure AWS account and Organization Unit
   - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/1_LI.jpg)
  - Create an AWS account (ignore if you already have one)
  - Create an Organization Unit 
  - Click 'Add an AWS account' and create a new AWS account from there with name as DevOps (you'll need another email address)
   ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/1_LI%20(2).jpg)
  - Login to the newly created account 
- Create a free domain name from http://www.freenom.com
![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/3.PNG)

- Create a hosted zone in AWS Route 53
  - Go to the Route 53 Console
  - Click 'Create Hosted Zone'
  - For Domain name, enter the domain name you got from freenom
 
 

  - Click on the created hosted zone and copy the contents of the NS record
  - Click 'Manage Domain' next to your domain name, and click Management Tools and select Nameservers
  
    ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/freenom.PNG)


## Step 1: Setup a Virtual Private Cloud

- Create a VPC from the VPC Management Console use a large enough CIDR block (/16)
- Create subnets as shown in the diagram above
     ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/4.PNG)
     
 
- Create a route table and associate it with the public subnets & private subnets
    ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/5.PNG)

- Create an Internet Gateway, select it and click Actions the click Attach to VPC and attach it to the VPC you created

  ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/6.PNG)
  
- Create a NAT Gateway for your private subnets (create one in each public subnet)
- ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/7.PNG)


- Allocate three Elastic IPs and attach one of them to the NAT Gateway

![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/8.PNG)


- Create a security group for:
  - Nginx servers: Access to nginx servers should only be from the Application Load Balancer
  - Bastion servers: Access to bastion servers should only be from the IPs of your workstation
  - Application Load Balancer: ALB should be open to the internet
  - Webservers: Webservers should only be accessible from the Nginx servers
  - Data Layer: This comprises the RDS and EFS servers. Access to RDS should only be from Webservers, while Nginx and Webservers can have access to EFS
  - 
![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/9.PNG)

## Step 2: Proceed with Compute Resources
### Step 2.1: Setup Compute Resources for Nginx
- Provision EC2 Instances for Nginx, Bastion and Webserver and install the necessary packages
  - Create 3 t2.micro RHEL 8 instance in any of your two public AZs
 
  - Create an AMIs of each instance created
  - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/10.PNG)
   
- Configure Target Groups for the 3 instatces
  - Enter the target group name
  - Select the VPC you created
  - For health checks, select HTTPS and health check path as /healthstatus
 - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/11.PNG)
 
 
- Configure Launch templates for tooling,bastion nginx and wordpress
  - Select the VPC and select the two public  and private subnets you created and configure respectively
  - For health checks, select ELB too. Click Next.
  - For Group size, enter 2 for minimum and desired capacity, 4 as maximum capacity
  - For Scaling policies, select Target Tracking scaling policy and set the target value as 90
  - Click Next and add Notifications, create a new SNS topic and enter your email under 'With these recipients'
   
 - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/18.PNG)
 - 

### Step 2.5: TLS Certificates from Amazon Certificate Manager (ACM)
- Navigate to AWS ACM
- Under 'Provision certificates' click Get started
- Click Request a certificate
- Enter the domain name you registered (*.\<domain-name>.com), click next
- Select DNS validation and click Next
- Tag the certificate, click Review then confirm and request
- Click Continue
- Click 'Export DNS Configuration file'
- Go to Route 53
- Create a new CNAME record with items from the DNS configuration.csv file downloaded.
- Give a few seconds for validation to complete
 
 - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/12.PNG)


### Step 2.4: Configure  Load Balancers 
 - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/lb.PNG)
 - Ensured to create the Internal(internal) and external(internet facing) load balancers within the same VPC i created 
 - Configured the required target groups for each load balancer
 -  ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/lbinternal.PNG)



## Step 3: Setup EFS
- Navigate to EFS from your Management Console
- Click create file system from the right
- Click Customize
- Enter the name for the EFS
- Tag the resource
- Leave everything else and click next
- Select the VPC you created, select the two AZs and choose the private subnets
- Select the EFS security group for each AZ
 - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/13.PNG)


## Step 4: Setup RDS
### Step 4.1: Create a KMS key
- Navigate to AWS KMS
- Click create key
- Make sure it's symmetric
- Give the key an alias
- For 'Define Key admininstrative privileges', select AWSServiceRoleForRDS and OrganizationAccountAccessRole 
-- ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/kms.PNG)

### Step 4.2: Create a DB Subnet Group
- Navigate to RDS Management Console
- Click the three horizontal lines on the top left
- Select Subnet groups
- Click Create DB subnet group
- Enter the name, description and select your VPC
- Under Add subnets, select the two AZs your data layer subnets are in and select the two private data layer subnets.
- Click Create
-- ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/rdssubnet.PNG)

### Step 4.3: Create RDS Instance
- Navigate to RDS Management Console
- Click Create database
- For Engine options, select MySQL
- For Template, choose Free tier
- Enter a name for your DB under DB instance identifier
- Enter Master username and passsword
- Select your VPC, select the subnet group you created and also the data layer security group
- ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/16.PNG)

## Step 5: Tie Everything Together
### Step 5.1: Configure DNS with Route 53
- Create a Alias record that points www.domain.com to the DNS name of your NGINX load balancer
- Create a Alias record that points tooling.domain.com to the DNS name of your NGINX load balancer
- 
 - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/20.PNG)

 - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/servers.PNG)
# Blockers
-  Trying to troubleshoot why my domainname cannot be accesible from the browser  yet all the target groups are health as shown below 
-  - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/tgroup1.PNG)
-  - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/tgroup2.PNG)
-  - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/tgroup3.PNG)

   # Also when  i run `curl -L localhost` on the tooling and wordpress server i can see the login page for php and wordpress select language
   - Tooling
   - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/tooling.PNG)
    -Wordpress
    - ![alt text](https://github.com/Ellawangari/DevOps-project15/blob/main/imgs/wordpress.PNG)
