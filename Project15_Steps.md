


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