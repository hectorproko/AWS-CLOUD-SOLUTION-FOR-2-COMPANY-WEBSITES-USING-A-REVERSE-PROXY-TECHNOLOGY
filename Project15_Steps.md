


#### Step 1

I start by getting a domain `hracompany.ga` from `www.freenom.com`

**Creating Organizational Unit** (OU) called **Dev**  
* AWS Organizations > AWS accounts
	* Select root account > Actions > Organizational Unit - Create New

Created a new account called **DevOps**  
* AWS Organizations > AWS accounts > Add an AWS Account

Moved **DevOps** account to Organizational Unit **Dev**  
* Select DevOps > Actions > Move  

Created Organizational Unit **Dev**  and put Account **DevOps**  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/organizationalUnit.png)  

Login to newly created account **DevOps**  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/accountDevOps.png)  

#### Create VPC 
* VPC > Your VPCs > Create VPC
* VPC Settings
  * Name tag: HRA-VPC
  * IPv4 CIDR: `10.0.0.0/16`
  * DNS hostnames: Enabled

Creating Internet Gateway **HRA-Igw** attach to **HRA-VPC**  
* VPC > Internet Gateways > Create internet gateway

* VCP > Subnets > Create subnet
  * Public Subnets
    * **Public Subnet 1** HRA-public-subnet-1  
		`10.0.1.0/24` | East 1a  
	* **Public Subnet 2** HRA-public-subnet-2  
		`10.0.3.0/24` | East 1b  
  * Private Subnets
	* **Private Subnet  1** HRA-private-subnet-1  
	`10.0.2.0/24` | East 1a
	* **Private Subnet  2** HRA-private-subnet-2  
	`10.0.4.0/24` | East 1b
	* **Private Subnet  3** HRA-private-subnet-3  
	`10.0.5.0/24` | East 1a
	* **Private Subnet  4** HRA-private-subnet-4  
	`10.0.6.0/24` | East 1b  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/subnet.png)  

**Creating Route Tables**
* VPC > Route Tables > Create route table
  * Created Route Tables  
	* **HRA-public-rtb**  
    * **HRA-private-rtb**  

![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/routeTable.png) 

**Associate Subnets to Route Tables**  
* **HRA-public-rtb** to Public Subnets 
  ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/subnetAssociationPublicRTB.gif)	 
	
* **HRA-private-rtb** to Private Subnets  
  ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/subnetAssociationPrivateRTB.gif)  

Edit Public Route Table **HRA-public-rtb** (to target **HRA-Igw** Internet Gateway)  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/publicRTB_internetGateway.gif)  

**Create a NAT gateway**  
* VPC > Elastic IPs > Allocate Elastic IP Address  
	Allocate Elastic IP (Tag name **HRA-NAT**)  
	![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/NAT_ElasticIP.gif)  
	
* VPC > NAT Gateways > Create NAT gateway  
	Create the NAT in NAT Gateways (**HRA-NatGateway**) into **Public Subnet 1** and choose Elastic IP **HRA-NAT**  
	![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/NATcreation.gif)  

Edit Private Route Table **HRA-private-rtb** like so _dest:_ `0.0.0.0/0`  _target:_ **HRA-NatGateway**  
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY/main/images/privateRTB_NAT.gif)  

**Creating Security Groups**
* VPC > SECURITY > Security Groups > Create security group  
	
	HRA-ext-ALB | HTTP/S from anywhere  
	HRA-bastion | SSH from anywhere (ideally from our current IP)  
	HRA-nginx-reverse-proxy | HTTP/s from HRA-ext-ALB  
	HRA-int-ALB | HTTP/S from HRA-nginx-reverse-proxy  
	HRA-webserver | SSH from HRA-bastion , HTTP/S from HRA-int-ALB 
	HRA-datalayer | MYSQL/Aurora from HRA-bastion , NFS from HRA-webserver , MYSQL/Aurora from HRA-webserver  